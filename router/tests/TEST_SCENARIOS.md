# Router Test Scenarios

**Source:** Step 1h discovery document  
**Purpose:** Executable test cases for router validation  
**Date:** Jan 19, 2026

---

## Test Execution Overview

All test cases adapted from `discovery/Step_1h_Test_Scenarios.md`. Each test validates a specific router path and ensures correct behavior.

---

## Test Case 1: Both ESI & MA Open, Odd Segment (Tie-Break to ESI)

**Input:**
- `XSEG_ASSIGNED` = 5 (odd segment)
- `XQINSTYPE` = r3 (ESI)
- `ESI_OPEN` = 30
- `MA_OPEN` = 25

**Expected Output:**
- `XRANDOMPICK` = r1 (ESI)
- `ROUTER_STATUS` = r2 (SUCCESS_ESI)
- `ROUTER_ALLOCATION_REASON` = r1 (QUOTA_AVAILABLE)
- `ROUTE_BLOCK_INTERNAL` = r1 (ROI_ESI_FINAL)
- `ROUTER_DECISION_LOG` contains: `XSEG=5; XQINS=r3; ESI_OPEN=30; MA_OPEN=25; DECISION=ROUTE; ROUTE=r1; STATUS=r2`

**Validation:**
- Respondent routed to ESI study block
- Tie-break logic: `5 % 2 == 1` → ESI
- No errors or soft-terms

---

## Test Case 2: Both ESI & MA Open, Even Segment (Tie-Break to MA)

**Input:**
- `XSEG_ASSIGNED` = 6 (even segment)
- `XQINSTYPE` = r3 (ESI)
- `ESI_OPEN` = 30
- `MA_OPEN` = 25

**Expected Output:**
- `XRANDOMPICK` = r2 (MA)
- `ROUTER_STATUS` = r1 (SUCCESS_MA)
- `ROUTER_ALLOCATION_REASON` = r1 (QUOTA_AVAILABLE)
- `ROUTE_BLOCK_INTERNAL` = r2 (ROI_MA_FINAL)
- `ROUTER_DECISION_LOG` contains: `XSEG=6; XQINS=r3; ESI_OPEN=30; MA_OPEN=25; DECISION=ROUTE; ROUTE=r2; STATUS=r1`

**Validation:**
- Respondent routed to MA study block
- Tie-break logic: `6 % 2 == 0` → MA
- No errors or soft-terms

---

## Test Case 3: ESI Only Open

**Input:**
- `XSEG_ASSIGNED` = 7
- `XQINSTYPE` = r3 (ESI)
- `ESI_OPEN` = 25
- `MA_OPEN` = 0

**Expected Output:**
- `XRANDOMPICK` = r1 (ESI)
- `ROUTER_STATUS` = r2 (SUCCESS_ESI)
- `ROUTER_ALLOCATION_REASON` = r1 (QUOTA_AVAILABLE)
- `ROUTE_BLOCK_INTERNAL` = r1 (ROI_ESI_FINAL)
- `ROUTER_DECISION_LOG` contains: `XSEG=7; XQINS=r3; ESI_OPEN=25; MA_OPEN=0; DECISION=ROUTE; ROUTE=r1; STATUS=r2`

**Validation:**
- Respondent routed to ESI (MA full)
- No fallback to GLP1 (ESI available)

---

## Test Case 4: MA Only Open

**Input:**
- `XSEG_ASSIGNED` = 8
- `XQINSTYPE` = r1 (MA)
- `ESI_OPEN` = 0
- `MA_OPEN` = 20

**Expected Output:**
- `XRANDOMPICK` = r2 (MA)
- `ROUTER_STATUS` = r1 (SUCCESS_MA)
- `ROUTER_ALLOCATION_REASON` = r1 (QUOTA_AVAILABLE)
- `ROUTE_BLOCK_INTERNAL` = r2 (ROI_MA_FINAL)
- `ROUTER_DECISION_LOG` contains: `XSEG=8; XQINS=r1; ESI_OPEN=0; MA_OPEN=20; DECISION=ROUTE; ROUTE=r2; STATUS=r1`

**Validation:**
- Respondent routed to MA (ESI full)
- No fallback to GLP1 (MA available)

---

## Test Case 5: Both MA & ESI Full (Overquota)

**Input:**
- `XSEG_ASSIGNED` = 9
- `XQINSTYPE` = r3 (ESI)
- `ESI_OPEN` = 0
- `MA_OPEN` = 0
- `GLP1_OPEN` = 0

**Expected Output:**
- `XRANDOMPICK` = UNSET (no allocation)
- `ROUTER_STATUS` = r4 (OVERQUOTA_NO_ALLOCATION)
- `ROUTER_ALLOCATION_REASON` = r2 (QUOTA_FULL)
- `ROUTE_BLOCK_INTERNAL` = r4 (NONE)
- `ROUTER_DECISION_LOG` contains: `XSEG=9; XQINS=r3; ESI_OPEN=0; MA_OPEN=0; GLP1_OPEN=0; DECISION=OVERQUOTA; ROUTE=r4; STATUS=r4`

**Validation:**
- Respondent shown soft-term message (Section 5, block `b_router_softterm`)
- **NOT hard-terminated** (no `<term>` node)
- Respondent record remains in Dynata as eligible for future invites
- No status code `rst=3` sent to panel

---

## Test Case 6: XQINSTYPE=Other (No MA/ESI Eligibility)

**Input:**
- `XSEG_ASSIGNED` = 10
- `XQINSTYPE` = r4 (Other)
- `ESI_OPEN` = INF
- `MA_OPEN` = INF
- `GLP1_OPEN` = 0

**Expected Output:**
- `XRANDOMPICK` = UNSET (not eligible for MA/ESI)
- `ROUTER_STATUS` = r4 (OVERQUOTA_NO_ALLOCATION)
- `ROUTER_ALLOCATION_REASON` = r3 (XQINSTYPE_OTHER_NO_ROUTE)
- `ROUTE_BLOCK_INTERNAL` = r4 (NONE)
- `ROUTER_DECISION_LOG` contains: `XSEG=10; XQINS=r4; ESI_OPEN=0; MA_OPEN=0; GLP1_OPEN=0; DECISION=OVERQUOTA; ROUTE=r4; STATUS=r4`

**Validation:**
- Respondent NOT routed to MA/ESI (insurance type r4 = Other)
- Soft-term because GLP1 also full
- Validates insurance eligibility logic (Step 1d)

---

## Test Case 7: XSEG_ASSIGNED Missing (Typing Incomplete)

**Input:**
- `XSEG_ASSIGNED` = NULL (typing failed)
- `XQINSTYPE` = r3 (ESI)
- `ESI_OPEN` = 25
- `MA_OPEN` = 20

**Expected Output:**
- `ROUTER_STATUS` = r5 (TYPING_INCOMPLETE)
- `ROUTER_ALLOCATION_REASON` = r4 (TYPING_FAILED)
- `ROUTE_BLOCK_INTERNAL` = r4 (NONE)
- `ROUTER_DECISION_LOG` contains: `XSEG=NULL; DECISION=ERROR; STATUS=r5`

**Validation:**
- Router exits early (Section 2 verification block)
- Soft-term with error message
- No allocation attempted

---

## Test Case 8: QINSTYPE=r99 (Invalid Insurance Type)

**Input:**
- `XSEG_ASSIGNED` = 11
- `QINSTYPE` = r99 (Not answered)
- `XQINSTYPE` = INVALID

**Expected Output:**
- `ROUTER_STATUS` = r6 (ERROR_INVALID_INSURANCE)
- `ROUTER_DECISION_LOG` contains: `XSEG=11; QINSTYPE=r99; DECISION=SCREENOUT; STATUS=r6`

**Validation:**
- Hard screenout via `term_QINSTYPE` (existing XML behavior)
- Router does NOT attempt allocation
- Validates existing termination logic preserved (Step 1e)

---

## Test Case 9: MA Flag, ESI Marginal (Boundary Case)

**Input:**
- `XSEG_ASSIGNED` = 12 (even segment)
- `XQINSTYPE` = r1 (MA)
- `ESI_OPEN` = 1 (marginal)
- `MA_OPEN` = 50

**Expected Output:**
- `XRANDOMPICK` = r2 (MA)
- `ROUTER_STATUS` = r1 (SUCCESS_MA)
- `ROUTER_ALLOCATION_REASON` = r1 (QUOTA_AVAILABLE)
- `ROUTE_BLOCK_INTERNAL` = r2 (ROI_MA_FINAL)
- `ROUTER_DECISION_LOG` contains: `XSEG=12; XQINS=r1; ESI_OPEN=1; MA_OPEN=50; DECISION=ROUTE; ROUTE=r2; STATUS=r1`

**Validation:**
- Both ESI & MA open (ESI=1 still counts as "open")
- Tie-break: `12 % 2 == 0` → MA
- Validates boundary condition (ESI at minimum)

---

## Test Case 10: Traditional Medicare (XQINSTYPE=r2)

**Input:**
- `XSEG_ASSIGNED` = 13
- `XQINSTYPE` = r2 (Traditional Medicare)
- `ESI_OPEN` = 0
- `MA_OPEN` = 30

**Expected Output:**
- `XRANDOMPICK` = r2 (MA)
- `ROUTER_STATUS` = r1 (SUCCESS_MA)
- `ROUTER_ALLOCATION_REASON` = r1 (QUOTA_AVAILABLE)
- `ROUTE_BLOCK_INTERNAL` = r2 (ROI_MA_FINAL)
- `ROUTER_DECISION_LOG` contains: `XSEG=13; XQINS=r2; ESI_OPEN=0; MA_OPEN=30; DECISION=ROUTE; ROUTE=r2; STATUS=r1`

**Validation:**
- XQINSTYPE=r2 (Traditional Medicare) eligible for MA study
- Validates insurance flag logic: `ins_ma_flag = (xqins in ["r1", "r2"])`
- Routed to MA successfully

---

## Test Execution Checklist

- [ ] Configure test environment with controlled quota states
- [ ] Run Test Case 1: Both open, odd segment → verify ESI allocation
- [ ] Run Test Case 2: Both open, even segment → verify MA allocation
- [ ] Run Test Case 3: ESI only → verify ESI allocation
- [ ] Run Test Case 4: MA only → verify MA allocation
- [ ] Run Test Case 5: All full → **verify soft-term (NO hard termination)**
- [ ] Run Test Case 6: XQINSTYPE=Other → verify no MA/ESI route
- [ ] Run Test Case 7: Typing incomplete → verify error handling
- [ ] Run Test Case 8: QINSTYPE=r99 → verify hard screenout
- [ ] Run Test Case 9: Boundary case → verify tie-break with ESI=1
- [ ] Run Test Case 10: Traditional Medicare → verify MA eligibility
- [ ] Export all `ROUTER_DECISION_LOG` values → verify audit trail accuracy
- [ ] Check Decipher quota decrements → verify counts update correctly
- [ ] Validate no Dynata `rst=3` codes on overquota → verify soft-term only

---

## Automated Testing (Future)

For production hardening, consider:
- Mock Dynata panel API responses
- Automated quota state manipulation
- Batch test execution (100+ respondents)
- Regression testing on router updates

**MVP scope:** Manual testing of all 10 scenarios sufficient for Jan 24 go-live.

---

## Test Results Template

**Test Date:** ___________  
**Tester:** ___________  
**Environment:** Staging / Production

| Test # | Scenario | Pass/Fail | Notes |
|--------|----------|-----------|-------|
| 1 | Both open, odd seg | ☐ Pass ☐ Fail | |
| 2 | Both open, even seg | ☐ Pass ☐ Fail | |
| 3 | ESI only | ☐ Pass ☐ Fail | |
| 4 | MA only | ☐ Pass ☐ Fail | |
| 5 | All full (soft-term) | ☐ Pass ☐ Fail | **Critical: verify no hard term** |
| 6 | XQINSTYPE=Other | ☐ Pass ☐ Fail | |
| 7 | Typing incomplete | ☐ Pass ☐ Fail | |
| 8 | QINSTYPE=r99 | ☐ Pass ☐ Fail | |
| 9 | Boundary (ESI=1) | ☐ Pass ☐ Fail | |
| 10 | Traditional Medicare | ☐ Pass ☐ Fail | |

**Overall Result:** ☐ All Pass → Ready for production  
**Issues Found:** ________________________

---

## Next Steps After Testing

1. Document any failures or unexpected behavior
2. Adjust router logic if needed (see `DESIGN_DECISIONS.md` for assumptions)
3. Retest failed cases
4. Get Bryan's sign-off
5. Deploy to production (target: Jan 24)
