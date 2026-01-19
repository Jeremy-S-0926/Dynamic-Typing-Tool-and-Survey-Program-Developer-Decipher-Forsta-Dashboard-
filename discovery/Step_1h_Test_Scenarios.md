# Step 1h: Test Scenarios & Validation Cases

**Status:** Complete (Jan 20, 2026)  
**Deliverable:** 10 test scenarios covering all router paths and edge cases

---

## Test Scenario Overview

| #   | Scenario                | Type            | XSEG_ASSIGNED | XQINSTYPE       | ESI_OPEN | MA_OPEN | Expected XRANDOMPICK | Expected ROUTER_STATUS  | Notes                       |
| --- | ----------------------- | --------------- | ------------- | --------------- | -------- | ------- | -------------------- | ----------------------- | --------------------------- |
| 1   | Both open, odd segment  | Happy path      | 5 (odd)       | 2 (ESI)         | 30       | 25      | 1 (ESI)              | SUCCESS_ESI             | Tie-break: mod-based        |
| 2   | Both open, even segment | Happy path      | 6 (even)      | 2 (ESI)         | 30       | 25      | 2 (MA)               | SUCCESS_MA              | Tie-break: mod-based        |
| 3   | ESI only open           | Happy path      | 7             | 2 (ESI)         | 25       | 0       | 1 (ESI)              | SUCCESS_ESI             | MA full; ESI available      |
| 4   | MA only open            | Happy path      | 8             | 0 (MA)          | 0        | 20      | 2 (MA)               | SUCCESS_MA              | ESI full; MA available      |
| 5   | Both full               | Overquota       | 9             | 2 (ESI)         | 0        | 0       | UNSET                | OVERQUOTA_NO_ALLOCATION | No quota available          |
| 6   | XQINSTYPE=Other         | Overquota       | 10            | 3 (Other)       | INF      | INF     | UNSET                | OVERQUOTA_NO_ALLOCATION | Other type not routable     |
| 7   | XSEG_ASSIGNED missing   | Error           | NULL          | 2 (ESI)         | 25       | 20      | UNSET                | TYPING_INCOMPLETE       | Typing module failed        |
| 8   | QINSTYPE=r99            | Error           | 11            | INVALID         | N/A      | N/A     | UNSET                | ERROR_INVALID_INSURANCE | Hard screenout              |
| 9   | MA flag, ESI marginal   | Boundary        | 12 (even)     | 0 (MA)          | 1        | 50      | 2 (MA)               | SUCCESS_MA              | ESI at 1 (edge case)        |
| 10  | Traditional Medicare    | Insurance logic | 13            | 1 (Traditional) | 0        | 30      | 2 (MA)               | SUCCESS_MA              | XQINSTYPE=1 eligible for MA |

---

## Detailed Test Cases

### TEST 1: Both ESI & MA Open, Odd Segment (Tie-Break to ESI)

**Setup:**
- Respondent completes typing: GOP/DEM scoring places them in segment 5
- Insurance: QINSTYPE=r1 (employer), XQINSTYPE=2 (ESI)
- Quota state: ESI_OPEN=30, MA_OPEN=25 (both available)
- Tie-break rule: XSEG_ASSIGNED % 2 = 5 % 2 = 1 (ESI)

**Execution:**
```
allocator reads Block_INSTYPE_Quota[2] (ESI):
  ESI_OPEN = min(Block.ESI_limit - current, Wave.ESI_limit - current) = 30
  MA_OPEN = min(Block.MA_limit - current, Wave.MA_limit - current) = 25

both > 0, so:
  XRANDOMPICK = (5 % 2) + 1 = 1 + 1 = 1 (nope, recalc)
  XRANDOMPICK = (5 % 2) = 1 (mod gives 1, add 1 if needed for clarity)
  
Actually: (XSEG_ASSIGNED % 2) gives 1 for odd, 0 for even.
  Add 1: odd (5) → (5 % 2) + 1 = 1 + 1 = 2 (MA)? 
  
Clarify: Let's use:
  XRANDOMPICK = 1 if (XSEG_ASSIGNED % 2 == 1) else 2
  → 5 % 2 = 1 (odd) → XRANDOMPICK = 1 (ESI) ✓
```

**Expected Output:**
- XRANDOMPICK=1
- ROUTE_BLOCK_INTERNAL=ROI_ESI_FINAL
- ROUTER_STATUS=SUCCESS_ESI
- ROUTER_DECISION_LOG contains: `XSEG=5; XQINS=2; ESI_OPEN=30; MA_OPEN=25; DECISION=ROUTE_TIE; ROUTE=ROI_ESI_FINAL`

**Validation:**
- ✅ Respondent routed to ESI block (b1)
- ✅ ROUTER_STATUS exported as SUCCESS_ESI
- ✅ ROUTER_DECISION_LOG audit trail readable

---

### TEST 2: Both ESI & MA Open, Even Segment (Tie-Break to MA)

**Setup:**
- Respondent completes typing: segment 6
- Insurance: QINSTYPE=r1 (employer), XQINSTYPE=2 (ESI)
- Quota state: ESI_OPEN=30, MA_OPEN=25

**Execution:**
```
XSEG = 6 (even)
6 % 2 = 0 (even) → XRANDOMPICK = 2 (MA) ✓
```

**Expected Output:**
- XRANDOMPICK=2
- ROUTE_BLOCK_INTERNAL=ROI_MA_FINAL
- ROUTER_STATUS=SUCCESS_MA

**Validation:**
- ✅ Respondent routed to MA block (b3)
- ✅ Deterministic: segment 6 always routes to MA (reproducible)

---

### TEST 3: ESI Only Open (MA Full)

**Setup:**
- Segment 7, XQINSTYPE=2 (ESI)
- Quota state: ESI_OPEN=25, MA_OPEN=0 (MA full)

**Execution:**
```
MA_OPEN = 0, so cannot allocate MA.
ESI_OPEN = 25 > 0, so:
  XRANDOMPICK = 1 (ESI)
  ROUTE_BLOCK = ROI_ESI_FINAL
  ROUTER_STATUS = SUCCESS_ESI
```

**Expected Output:**
- XRANDOMPICK=1
- ROUTER_STATUS=SUCCESS_ESI
- ROUTER_ALLOCATION_REASON=QUOTA_AVAILABLE (ESI only)

**Validation:**
- ✅ Respondent allocated to ESI despite tie-break rule (ESI only available)
- ✅ Decision logged as forced allocation, not tie-break

---

### TEST 4: MA Only Open (ESI Full)

**Setup:**
- Segment 8, XQINSTYPE=0 (Medicare Advantage)
- Quota state: ESI_OPEN=0 (full), MA_OPEN=20 (available)

**Execution:**
```
ESI_OPEN = 0 (no ESI available)
MA_OPEN = 20 > 0, so:
  XRANDOMPICK = 2 (MA)
  ROUTE_BLOCK = ROI_MA_FINAL
  ROUTER_STATUS = SUCCESS_MA
```

**Expected Output:**
- XRANDOMPICK=2
- ROUTER_STATUS=SUCCESS_MA

**Validation:**
- ✅ Respondent with MA eligibility allocated to MA
- ✅ No forced soft-term (MA available)

---

### TEST 5: Both Quotas Full (Overquota Soft-Term)

**Setup:**
- Segment 9, XQINSTYPE=2 (ESI)
- Quota state: ESI_OPEN=0, MA_OPEN=0 (both full)

**Execution:**
```
ESI_OPEN = 0
MA_OPEN = 0
→ no allocation possible
→ XRANDOMPICK = UNSET
→ ROUTER_STATUS = OVERQUOTA_NO_ALLOCATION
→ ROUTER_ALLOCATION_REASON = QUOTA_FULL
```

**Expected Output:**
- XRANDOMPICK=NULL
- ROUTER_STATUS=OVERQUOTA_NO_ALLOCATION
- ROUTER_ALLOCATION_REASON=QUOTA_FULL
- Respondent routed to b_softterm block (soft-term message)

**Validation:**
- ✅ Soft-term message displayed (no hard exit code)
- ✅ ROUTER_STATUS exported (for recovery if quotas reopen)
- ✅ No Dynata status code written (respondent remains eligible if quotas refresh)

---

### TEST 6: XQINSTYPE=Other (No Route Available)

**Setup:**
- Segment 10, XQINSTYPE=3 (Other insurance type)
- Quota state: ESI_OPEN=100, MA_OPEN=100 (both available)

**Execution:**
```
INS_OTHER_FLAG = (XQINSTYPE == 3) = TRUE
→ Other type not routable to MA/ESI
→ ROUTER_STATUS = OVERQUOTA_NO_ALLOCATION (or ERROR_OTHER_NO_ROUTE)
→ ROUTER_ALLOCATION_REASON = XQINSTYPE_OTHER_NO_ROUTE
```

**Expected Output:**
- XRANDOMPICK=NULL
- ROUTER_STATUS=OVERQUOTA_NO_ALLOCATION
- ROUTER_ALLOCATION_REASON=XQINSTYPE_OTHER_NO_ROUTE

**Validation:**
- ✅ Respondent soft-termed (no MA/ESI route for Other)
- ✅ Decision logged (not a quota issue, but ineligible type)
- ✅ Respondent can be recovered later if Other routing defined

---

### TEST 7: Typing Incomplete (XSEG_ASSIGNED Missing)

**Setup:**
- XSEG_ASSIGNED=NULL (typing module failed to complete)
- XQINSTYPE=2 (insurance answered)
- Quota state: both open

**Execution:**
```
if (XSEG_ASSIGNED == NULL):
  → cannot proceed to allocation
  → ROUTER_STATUS = TYPING_INCOMPLETE
  → XRANDOMPICK = UNSET
```

**Expected Output:**
- XSEG_ASSIGNED=NULL
- ROUTER_STATUS=TYPING_INCOMPLETE
- Respondent routed to soft-term or error block

**Validation:**
- ✅ Router detects incomplete typing
- ✅ Error logged (not quota issue)
- ✅ Respondent soft-termed with explanatory message

---

### TEST 8: Insurance Screenout (QINSTYPE=r99)

**Setup:**
- QINSTYPE=r99 (Not answered)
- Typing completed (segment assigned)

**Execution:**
```
QINSTYPE validation (before XQINSTYPE derivation):
if (QINSTYPE == r99):
  → hard screenout (existing system logic)
  → ROUTER_STATUS = ERROR_INVALID_INSURANCE
  → route to term node (with screenout marker)
```

**Expected Output:**
- ROUTER_STATUS=ERROR_INVALID_INSURANCE
- Respondent hard-terminated (not soft-term)
- Dynata status code written (existing PRISM logic)

**Validation:**
- ✅ Preserves existing hard-screenout behavior (no change)
- ✅ Error logged in ROUTER_STATUS for audit
- ✅ Router defers to existing term node

---

### TEST 9: Marginal ESI Quota (Boundary Case)

**Setup:**
- Segment 12 (even, normally MA), XQINSTYPE=0 (Medicare Advantage)
- Quota state: ESI_OPEN=1 (last slot), MA_OPEN=50 (plenty)
- Medicare respondents eligible for both MA & ESI per XQINSTYPE eligibility rules

**Execution:**
```
# Wait: XQINSTYPE=0 is Medicare Advantage (only MA eligible)
# Let's revise:

Setup:
- Segment 12 (even), XQINSTYPE=0 (MA only)
- ESI_OPEN=0, MA_OPEN=1

Execution:
  ESI_OPEN = 0
  MA_OPEN = 1 > 0
  → XRANDOMPICK = 2 (MA)
  → ROUTE_BLOCK = ROI_MA_FINAL
  → ROUTER_STATUS = SUCCESS_MA
  → ROUTER_ALLOCATION_REASON = QUOTA_AVAILABLE
```

**Expected Output:**
- XRANDOMPICK=2
- ROUTER_STATUS=SUCCESS_MA
- ROUTER_ALLOCATION_REASON=QUOTA_AVAILABLE

**Validation:**
- ✅ Respondent allocated even at marginal quota (1 slot left)
- ✅ Quota audit trail shows ESI_OPEN=0, MA_OPEN=1
- ✅ Respondent count incremented after completion (post-hoc quota refresh)

---

### TEST 10: Traditional Medicare (XQINSTYPE Derivation)

**Setup:**
- Segment 13, QINSTYPE=r7 (Medicare)
- QINS_MEDICARE=r1 (Traditional Medicare)
- Quota state: ESI_OPEN=10, MA_OPEN=40 (both available)

**Execution:**
```
XQINSTYPE derivation:
  if QINS_MEDICARE == r1:
    XQINSTYPE = 1 (Traditional Medicare)
  
INS_MA_FLAG = (XQINSTYPE in {0, 1}) = TRUE
INS_ESI_FLAG = (XQINSTYPE == 2) = FALSE

Allocation:
  ESI_OPEN = 0 (Traditional Medicare not ESI-eligible)
  MA_OPEN = 40 (Traditional Medicare eligible for MA)
  
  XRANDOMPICK = 2 (MA)
  ROUTE_BLOCK = ROI_MA_FINAL
  ROUTER_STATUS = SUCCESS_MA
```

**Expected Output:**
- XQINSTYPE=1
- XRANDOMPICK=2
- ROUTER_STATUS=SUCCESS_MA

**Validation:**
- ✅ Traditional Medicare respondents correctly routed to MA (not ESI)
- ✅ XQINSTYPE derivation matches documented logic
- ✅ Insurance eligibility flags enforced

---

## Cross-Test Validation

### Consistency Checks
- [ ] Tie-break rule consistent across all "both open" tests (odd→1/ESI, even→2/MA)
- [ ] Overquota status consistent (both full, Other type, incomplete typing all → OVERQUOTA_NO_ALLOCATION or ERROR_*)
- [ ] ROUTER_DECISION_LOG populated in all tests (audit trail complete)
- [ ] Soft-term tests (5, 6, 7) do NOT write Dynata status codes (rt field remains NULL)
- [ ] Hard-term test (8) writes Dynata status code (existing logic preserved)
- [ ] Export schema matches declared hidden variables (XSEG, XQINS, XRANDOMPICK, ROUTER_STATUS, ROUTER_DECISION_LOG)

### Edge Cases Covered
- ✅ Tie-break (tests 1, 2): alternating segments route deterministically
- ✅ Forced allocation (tests 3, 4): when only one option available
- ✅ Overquota (test 5): both quotas exhausted
- ✅ Ineligible type (test 6): XQINSTYPE=Other has no route
- ✅ Incomplete typing (test 7): early exit detection
- ✅ Screenout (test 8): existing hard-term preserved
- ✅ Marginal quota (test 9): boundary of availability
- ✅ Insurance derivation (test 10): Traditional Medicare logic

### Regression Tests (Existing PRISM Behavior)
- ✅ Hard screenouts still execute (QINSTYPE=r99 in test 8)
- ✅ Typing module still completes (tests 1–6, 9–10)
- ✅ XSEG_ASSIGNED and XQINSTYPE variables still exported (all tests)
- ✅ Quota sheets still read live from Decipher (all tests)
- ✅ No changes to GOP/DEM scoring logic (tests validate downstream)

---

## Test Execution Checklist

### Pre-Test Setup
- [ ] XML stub created with router execs + decision tree
- [ ] Hidden variables declared (XRANDOMPICK, ROUTER_STATUS, ROUTER_DECISION_LOG, ROUTER_ALLOCATION_REASON)
- [ ] Soft-term message block configured (b_softterm)
- [ ] ESI block (b1) and MA block (b3) preserved unchanged
- [ ] Decipher quota sheets loaded (Block_INSTYPE_Quota, Wave One Equal Allocation)

### Per-Test
- [ ] Input respondent data correct (segment, insurance, quota state)
- [ ] XRANDOMPICK output matches tie-break rule
- [ ] ROUTER_STATUS matches expected decision
- [ ] ROUTER_DECISION_LOG truncates to <500 chars without loss of critical info
- [ ] Routing conditional correct (block cond="XRANDOMPICK == ..." triggers)
- [ ] Soft-term message displays (no hard exit)
- [ ] Data export includes all router variables

### Post-Test Integration
- [ ] All 10 tests pass
- [ ] No hard termination codes written in soft-term tests
- [ ] Respondent record remains recoverable if quotas reopen
- [ ] QA audit trail complete and parseable
- [ ] Performance: router completes <30 seconds per respondent

---

## Known Limitations & Future Work

| Issue | Status | Workaround | Future Fix |
|-------|--------|-----------|-----------|
| Tie-break currently deterministic (mod-based) | Current | Use segment ID for reproducibility | Implement configurable 50/50 marker-based random |
| No AL/VAX/PULSE quota sheets provided | Pending | Test with MA/ESI only | Obtain missing quota sheets from Bryan |
| Soft-term message hard-coded | Current | Customize per-study in Decipher | Implement message template system |
| ROUTER_DECISION_LOG truncated at 500 chars | Current | Prioritize key fields (XSEG, XQINS, DECISION) | Compress or use separate debug export |
| No real-time quota refresh during session | Current | Quotas read once at entry | Implement mid-session refresh if needed |

