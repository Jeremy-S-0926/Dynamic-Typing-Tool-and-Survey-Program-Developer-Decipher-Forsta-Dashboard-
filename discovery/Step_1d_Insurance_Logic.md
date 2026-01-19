# Step 1d: Insurance Classification Logic

## Insurance Questions & Logic (from PRISM_MA_ESI.xml, lines 885–940)

### Question 1: QINSTYPE (line 905)
**Label:** "What is the main type of health insurance coverage that you currently have?"

**Response options:**
- r1 (value=1): "A plan through my employer"
- r2 (value=2): "A plan through my spouse's employer"
- r3 (value=3): "A plan through my parent's employer"
- r4 (value=4): "A plan you purchased yourself directly from an insurance company"
- r7 (value=7): "Medicare"
- r99 (value=99): [Not answered; triggers screenout `term_QINSTYPE`]

**Screenout:** If r99 selected, respondent is terminated with marker `Screened-QINSTYPE` (line 921–928).

### Question 2: QINS_MEDICARE (line 933)
**Shown if:** `QINSTYPE.r7` (Medicare selected)

**Label:** "Do you have traditional or original Medicare, or do you have a Medicare Advantage plan?"

**Response options:**
- r1 (value=1): "Traditional Medicare"
- r2 (value=2): "Medicare Advantage"

---

## Derived Variable: XQINSTYPE (lines 2584–2602)

**Purpose:** Convert QINSTYPE + QINS_MEDICARE into a single insurance type classifier.

**Logic (Decipher pseudo-code):**
```
if QINS_MEDICARE.r2:
    XQINSTYPE = 0  // Medicare Advantage
elif QINS_MEDICARE.r1:
    XQINSTYPE = 1  // Traditional Medicare
elif QINSTYPE.r1 or QINSTYPE.r2 or QINSTYPE.r3:
    XQINSTYPE = 2  // Employer-Sponsored Insurance (ESI)
else:
    XQINSTYPE = 3  // Other
```

**Rows (for quota sheets):**
- r1: Medicare Advantage
- r2: Traditional Medicare
- r3: Employer Sponsored Insurance
- r4: Others

---

## MA vs ESI Branching Logic (lines 2620–2670)

### MA/ESI Random Picker: XRANDOMPICK

**Purpose:** When a respondent has both MA and ESI eligibility, randomly assign to one based on quota headroom.

**Input:** Block_INSTYPE_Quota sheet for the respondent's XQINSTYPE row.

**Logic:**
```
x = XQINSTYPE (0, 1, 2, or 3)
ESI_BLOCK = Block_INSTYPE_Quota[XQINSTYPE_{x+1}][ESI_SEC] limit - current
MA_BLOCK  = Block_INSTYPE_Quota[XQINSTYPE_{x+1}][MA_SEC] limit - current

if ESI_BLOCK > 0 AND MA_BLOCK > 0:
    // Both open; pick one via marker on Block Quota (50/50 or weighted)
    for each marker in ["/Block Quota/c_1", "/Block Quota/c_2"]:
        if hasMarker(marker):
            XRANDOMPICK = 0 or 1  // c_1=ESI, c_2=MA
elif ESI_BLOCK == 0 AND MA_BLOCK > 0:
    XRANDOMPICK = 1  // MA only
elif ESI_BLOCK > 0 AND MA_BLOCK == 0:
    XRANDOMPICK = 0  // ESI only
// (if both <= 0, XRANDOMPICK left unset; quotas full)
```

**Output rows:**
- r1: ESI
- r2: MA

---

## Implications for Router Design

### Insurance Eligibility Flags
For router implementation, derive these flags from XQINSTYPE:

| Flag | Condition | Meaning |
|------|-----------|---------|
| `INS_MA_FLAG` | XQINSTYPE ∈ {0, 1} | Medicare (MA or Traditional) |
| `INS_ESI_FLAG` | XQINSTYPE == 2 | Employer-sponsored |
| `INS_OTHER_FLAG` | XQINSTYPE == 3 | Other/no insurance |

### Study Routing Precedence
**Current (MA/ESI variant):**
- If INS_MA_FLAG: eligible for MA (higher priority when both open)
- If INS_ESI_FLAG: eligible for ESI
- If both: XRANDOMPICK determines assignment (see Block_INSTYPE_Quota + Block Quota picker logic above)

**For GLP1/AL/VAX variants:**
- No Medicare/ESI branching; all eligible regardless of XQINSTYPE.

---

## Open Questions
1. **XQINSTYPE r4 (Others):** Should "Other" respondents be routed to a specific fallback study or terminated? Currently no explicit routing defined.
2. **Block markers:** Are Block Quota c_1 / c_2 markers currently live in Decipher, or should we set deterministic 50/50 split?
3. **Existing INS_* flags:** Are any INS_MA/ESI flags already computed in existing XMLs, or only XQINSTYPE?

---

## How to Verify in VS Code
1. Search `label="QINSTYPE"` in `PRISM_MA_ESI.xml` to see insurance question.
2. Search `label="QINS_MEDICARE"` to see Medicare Advantage vs Traditional branching.
3. Search `label="XQINSTYPE"` to see the derived variable and exec logic.
4. Search `XRANDOMPICK` to see MA/ESI branch assignment logic.
5. Verify identical logic exists in `PRISM_GLP1` and `PRISM_AL_VAX` or note differences.
