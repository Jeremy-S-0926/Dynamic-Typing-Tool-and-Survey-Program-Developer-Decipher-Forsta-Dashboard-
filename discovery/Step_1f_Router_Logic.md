# Step 1f: Router Decision Tree & Pseudocode

**Status:** Complete (Jan 20, 2026)  
**Deliverable:** Full router algorithm for single-survey shell implementation

---

## Router Flowchart

```
┌─ ENTRY (Respondent loads survey link)
│
├─ TYPING MODULE
│  ├─ Run GOP/DEM scoring
│  ├─ Assign XSEG_ASSIGNED (1–16)
│  └─ If incomplete → ROUTER_STATUS=TYPING_INCOMPLETE; SOFT_TERM
│
├─ INSURANCE CLASSIFICATION
│  ├─ QINSTYPE (employer/self/Medicare) → verify not r99
│  ├─ QINS_MEDICARE (if r7) → Traditional or MA
│  ├─ Derive XQINSTYPE (0=MA, 1=Traditional, 2=ESI, 3=Other)
│  ├─ Derive INS_MA_FLAG = (XQINSTYPE ∈ {0,1})
│  ├─ Derive INS_ESI_FLAG = (XQINSTYPE == 2)
│  └─ If invalid → ROUTER_STATUS=ERROR_INVALID_INSURANCE; SOFT_TERM
│
├─ QUOTA ALLOCATOR
│  ├─ Read Block_INSTYPE_Quota[XQINSTYPE]
│  ├─ Read Wave One Equal Allocation[XSEG_ASSIGNED]
│  ├─ Compute available quotas:
│  │   ├─ if INS_MA_FLAG:
│  │   │   ESI_OPEN = min(Block_INSTYPE.ESI_SEC - current, Wave_ESI - current)
│  │   │   MA_OPEN = min(Block_INSTYPE.MA_SEC - current, Wave_MA - current)
│  │   ├─ elif INS_ESI_FLAG:
│  │   │   ESI_OPEN = min(Block_INSTYPE.ESI_SEC - current, Wave_ESI - current)
│  │   │   MA_OPEN = 0
│  │   └─ elif INS_OTHER_FLAG:
│  │       ESI_OPEN = 0; MA_OPEN = 0
│  │
│  ├─ TIE-BREAK: Pick allocation
│  │  ├─ if both ESI_OPEN > 0 AND MA_OPEN > 0:
│  │  │   ├─ XRANDOMPICK = 50/50 via marker (Block Quota c_1/c_2)
│  │  │   └─ if deterministic preferred: XRANDOMPICK = (XSEG_ASSIGNED % 2) ? 1 : 2
│  │  ├─ elif ESI_OPEN > 0:
│  │  │   └─ XRANDOMPICK = 1 (ESI only)
│  │  ├─ elif MA_OPEN > 0:
│  │  │   └─ XRANDOMPICK = 2 (MA only)
│  │  └─ else:
│  │      └─ XRANDOMPICK = UNSET; allocation = NONE
│  │
│  └─ ROUTE BLOCK ASSIGNMENT
│     ├─ if XRANDOMPICK == 1 (ESI):
│     │   └─ ROUTE_BLOCK = ROI_ESI_FINAL (block b1)
│     ├─ elif XRANDOMPICK == 2 (MA):
│     │   └─ ROUTE_BLOCK = ROI_MA_FINAL (block b3)
│     └─ else:
│         └─ ROUTE_BLOCK = NONE; go to SOFT_TERM
│
├─ ASSIGNMENT DECISION
│  ├─ if ROUTE_BLOCK = ROI_ESI_FINAL or ROI_MA_FINAL:
│  │   ├─ ROUTER_STATUS = SUCCESS_ESI or SUCCESS_MA
│  │   ├─ Write hidden variables (XSEG_ASSIGNED, XQINSTYPE, XRANDOMPICK, ROUTER_STATUS, ROUTER_DECISION_LOG)
│  │   └─ ROUTE → block b1 (ESI) or b3 (MA) → continue survey
│  │
│  └─ else (no allocation available):
│      ├─ ROUTER_STATUS = OVERQUOTA_NO_ALLOCATION
│      ├─ Write hidden variables
│      └─ SOFT_TERM (polite message)
│
└─ END
```

---

## Pseudocode (Implementation Ready)

```python
def route_respondent(respondent_data, quota_state):
    """
    Main router algorithm: typing → insurance → allocator → route/term
    
    Args:
        respondent_data: dict with responses from typing module + insurance questions
        quota_state: dict with live Block_INSTYPE_Quota + Wave Equal Allocation
    
    Returns:
        dict: {
            'XSEG_ASSIGNED': int 1-16,
            'XQINSTYPE': int 0-3,
            'XRANDOMPICK': int 1-2 or None,
            'ROUTE_BLOCK': str 'ROI_ESI_FINAL' | 'ROI_MA_FINAL' | None,
            'ROUTER_STATUS': str,
            'ROUTER_DECISION_LOG': str,
            'ROUTER_ALLOCATION_REASON': str
        }
    """
    
    # ──────────────────────────────────
    # STEP 1: TYPING MODULE
    # ──────────────────────────────────
    result = {
        'XSEG_ASSIGNED': None,
        'XQINSTYPE': None,
        'XRANDOMPICK': None,
        'ROUTE_BLOCK': None,
        'ROUTER_STATUS': None,
        'ROUTER_DECISION_LOG': '',
        'ROUTER_ALLOCATION_REASON': None
    }
    
    # Run GOP/DEM scoring (already done in XML; just verify XSEG_ASSIGNED exists)
    XSEG_ASSIGNED = respondent_data.get('XSEG_ASSIGNED')
    if not XSEG_ASSIGNED or XSEG_ASSIGNED < 1 or XSEG_ASSIGNED > 16:
        result['ROUTER_STATUS'] = 'TYPING_INCOMPLETE'
        return result  # SOFT_TERM
    
    result['XSEG_ASSIGNED'] = XSEG_ASSIGNED
    
    # ──────────────────────────────────
    # STEP 2: INSURANCE CLASSIFICATION
    # ──────────────────────────────────
    
    QINSTYPE = respondent_data.get('QINSTYPE')  # r1-r4, r7, r99
    QINS_MEDICARE = respondent_data.get('QINS_MEDICARE')  # r1 (Traditional) or r2 (MA)
    
    # Hard screenout: not answered
    if QINSTYPE == 'r99':
        result['ROUTER_STATUS'] = 'ERROR_INVALID_INSURANCE'
        return result  # HARD_TERM (existing logic)
    
    # Derive XQINSTYPE
    if QINS_MEDICARE == 'r2':
        # Medicare Advantage
        XQINSTYPE = 0
    elif QINS_MEDICARE == 'r1':
        # Traditional Medicare
        XQINSTYPE = 1
    elif QINSTYPE in ('r1', 'r2', 'r3'):
        # Employer-sponsored (self, spouse, parent)
        XQINSTYPE = 2
    else:
        # Other (r4) or undefined
        XQINSTYPE = 3
    
    result['XQINSTYPE'] = XQINSTYPE
    
    # Derive eligibility flags
    INS_MA_FLAG = (XQINSTYPE in (0, 1))    # Medicare (MA or Traditional)
    INS_ESI_FLAG = (XQINSTYPE == 2)        # Employer-sponsored
    INS_OTHER_FLAG = (XQINSTYPE == 3)      # Other
    
    # If Other, no allocation available in current system
    if INS_OTHER_FLAG:
        result['ROUTER_STATUS'] = 'OVERQUOTA_NO_ALLOCATION'
        result['ROUTER_ALLOCATION_REASON'] = 'XQINSTYPE_OTHER_NO_ROUTE'
        return result  # SOFT_TERM
    
    # ──────────────────────────────────
    # STEP 3: QUOTA ALLOCATOR
    # ──────────────────────────────────
    
    # Read quota constraints for this XQINSTYPE
    block_quota = quota_state['Block_INSTYPE_Quota'][XQINSTYPE]
    wave_quota = quota_state['Wave One Equal Allocation'][XSEG_ASSIGNED]
    
    # Compute available quota headroom
    ESI_BLOCK_AVAILABLE = max(0, block_quota['ESI_SEC_limit'] - block_quota['ESI_SEC_current'])
    MA_BLOCK_AVAILABLE = max(0, block_quota['MA_SEC_limit'] - block_quota['MA_SEC_current'])
    
    ESI_WAVE_AVAILABLE = max(0, wave_quota['ESI_limit'] - wave_quota['ESI_current'])
    MA_WAVE_AVAILABLE = max(0, wave_quota['MA_limit'] - wave_quota['MA_current'])
    
    # Combined available (bottleneck is minimum of block and wave)
    ESI_OPEN = min(ESI_BLOCK_AVAILABLE, ESI_WAVE_AVAILABLE) if INS_MA_FLAG or INS_ESI_FLAG else 0
    MA_OPEN = min(MA_BLOCK_AVAILABLE, MA_WAVE_AVAILABLE) if INS_MA_FLAG else 0
    
    # ──────────────────────────────────
    # STEP 4: TIE-BREAK & ROUTE SELECTION
    # ──────────────────────────────────
    
    XRANDOMPICK = None
    ROUTE_BLOCK = None
    allocation_reason = None
    
    if INS_MA_FLAG and ESI_OPEN > 0 and MA_OPEN > 0:
        # Both blocks open: use marker-based 50/50 or deterministic selector
        marker = quota_state.get('Block_Quota_marker', 'c_1')  # c_1 or c_2 from Block Quota sheet
        
        # Option A: Deterministic tie-break (mod-based on segment)
        XRANDOMPICK = (XSEG_ASSIGNED % 2) + 1  # 1 or 2
        
        # Option B: Use marker (50/50 split, may require real marker from sheet)
        # XRANDOMPICK = 1 if marker == 'c_1' else 2
        
        ROUTE_BLOCK = 'ROI_ESI_FINAL' if XRANDOMPICK == 1 else 'ROI_MA_FINAL'
        allocation_reason = 'QUOTA_AVAILABLE'
    
    elif ESI_OPEN > 0:
        # ESI only available
        XRANDOMPICK = 1
        ROUTE_BLOCK = 'ROI_ESI_FINAL'
        allocation_reason = 'QUOTA_AVAILABLE'
    
    elif MA_OPEN > 0:
        # MA only available (requires INS_MA_FLAG)
        XRANDOMPICK = 2
        ROUTE_BLOCK = 'ROI_MA_FINAL'
        allocation_reason = 'QUOTA_AVAILABLE'
    
    else:
        # No quota available
        result['ROUTER_STATUS'] = 'OVERQUOTA_NO_ALLOCATION'
        result['ROUTER_ALLOCATION_REASON'] = 'QUOTA_FULL'
        result['ROUTER_DECISION_LOG'] = (
            f"{timestamp()}; XSEG={XSEG_ASSIGNED}; XQINS={XQINSTYPE}; "
            f"ESI_OPEN={ESI_OPEN}; MA_OPEN={MA_OPEN}; DECISION=OVERQUOTA"
        )
        return result  # SOFT_TERM
    
    # ──────────────────────────────────
    # STEP 5: SUCCESS ASSIGNMENT
    # ──────────────────────────────────
    
    result['XRANDOMPICK'] = XRANDOMPICK
    result['ROUTE_BLOCK'] = ROUTE_BLOCK
    result['ROUTER_STATUS'] = 'SUCCESS_' + ('ESI' if XRANDOMPICK == 1 else 'MA')
    result['ROUTER_ALLOCATION_REASON'] = allocation_reason
    result['ROUTER_DECISION_LOG'] = (
        f"{timestamp()}; XSEG={XSEG_ASSIGNED}; XQINS={XQINSTYPE}; "
        f"INS_MA={INS_MA_FLAG}; INS_ESI={INS_ESI_FLAG}; "
        f"ESI_OPEN={ESI_OPEN}; MA_OPEN={MA_OPEN}; "
        f"DECISION=ROUTE; ROUTE={ROUTE_BLOCK}; STATUS={result['ROUTER_STATUS']}"
    )
    
    return result
```

---

## Allocation Priority Precedence

### When Multiple Routes Available:
1. **Both MA & ESI open, INS_MA_FLAG:** Use mod-based deterministic tie-break (XSEG_ASSIGNED % 2)
   - Odd segments → ESI (XRANDOMPICK=1)
   - Even segments → MA (XRANDOMPICK=2)
   - *Alternative:* Use marker from Block Quota sheet for 50/50 random

2. **ESI only open:** Allocate to ESI (XRANDOMPICK=1)

3. **MA only open, INS_MA_FLAG:** Allocate to MA (XRANDOMPICK=2)

4. **No quota open:** Soft-term (ROUTER_STATUS=OVERQUOTA_NO_ALLOCATION)

### Deterministic Tie-Break Justification
- **Stability:** Same respondent segment always routes same way (reproducible)
- **Panel balance:** Alternating odd/even ensures roughly 50/50 split without randomness
- **Audit trail:** Easy to verify why a segment was routed (XSEG_ASSIGNED % 2)
- **Fallback to marker:** If Bryan prefers 50/50 random, use Block Quota marker instead

---

## Hidden Variable Assignment

After router completes, write to Decipher hidden fields:

| Variable | Value | Purpose |
|----------|-------|---------|
| `XSEG_ASSIGNED` | 1–16 | Final segment (already set by typing) |
| `XQINSTYPE` | 0–3 | Insurance class (already set in current system) |
| `XRANDOMPICK` | 1 or 2 | MA/ESI selector (new: router-driven) |
| `ROUTER_STATUS` | string | PRIMARY decision flag (SUCCESS_*, OVERQUOTA_*, ERROR_*) |
| `ROUTER_DECISION_LOG` | string | Audit trail (timestamp, quota states, decision) |
| `ROUTER_ALLOCATION_REASON` | string | Why allocated/not allocated |

---

## Quota Read Timing

**When to read quotas:**
- At router entry (after insurance classification, before allocator)
- Read live from Decipher `Block_INSTYPE_Quota` + `Wave One Equal Allocation` sheets
- Quota state may change mid-survey (other respondents completing); use most recent read

**Frequency:**
- Once per respondent entry (single read)
- No periodic refresh during single survey session

---

## Error Handling

| Error | Code | Action |
|-------|------|--------|
| XSEG_ASSIGNED missing | TYPING_INCOMPLETE | Soft-term with message |
| QINSTYPE = r99 | ERROR_INVALID_INSURANCE | Hard-term (existing logic) |
| All quotas full | OVERQUOTA_NO_ALLOCATION | Soft-term with message |
| XQINSTYPE = Other | OVERQUOTA_NO_ALLOCATION | Soft-term (no MA/ESI route) |
| Quota read failure | ERROR_QUOTA_READ | Log to QA; treat as overquota |

---

## Testing Checklist

- [ ] Test Scenario 1: Both ESI & MA open → tie-break selects ESI (odd segment)
- [ ] Test Scenario 2: Both ESI & MA open → tie-break selects MA (even segment)
- [ ] Test Scenario 3: ESI only → allocate to ESI
- [ ] Test Scenario 4: MA only → allocate to MA
- [ ] Test Scenario 5: Both full → soft-term
- [ ] Test Scenario 6: XQINSTYPE=Other → soft-term (no route)
- [ ] Test Scenario 7: XSEG_ASSIGNED missing → soft-term
- [ ] Test Scenario 8: QINSTYPE=r99 → hard-term
