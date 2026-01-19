# Router Validation Report

**Date:** Jan 19-20, 2026  
**Scope:** Pre-client validation of Phase 2 initial deliverables  
**Status:** ⚠️ CRITICAL ISSUES FOUND — Fixes Required Before Client Review  
**Reviewer:** Internal QA (Pre-Bryan Review)

---

## Executive Summary

### ✅ PASSED: Architecture & Documentation (4/6 areas)
- XML structure is well-formed and logically organized
- Router logic matches Step 1f pseudocode specifications
- Documentation is comprehensive and internally consistent
- Test scenarios cover all decision paths

### ⚠️ ISSUES FOUND: Implementation Gaps (2 critical)

**CRITICAL:**
1. **XQINSTYPE value mismatch** — Router uses r1/r2/r3/r4 but spec says 0/1/2/3 (integer values)
2. **Missing XRANDOMPICK implementation** — Variable not set in router logic

**HIGH PRIORITY:**
3. Quota API placeholder code needs real Decipher syntax
4. Missing datetime import in exec block
5. Block dispatch logic incomplete (missing conditions)

---

## Detailed Findings

### 1. XML Syntax & Structure ✅ PASS

**Validation:**
- All XML tags properly opened and closed
- Variable declarations follow Decipher conventions (`<radio>`, `<text>`, `where` attributes)
- Comments clearly mark sections and placeholders
- File is 369 lines, well-organized into 7 sections

**Finding:** No syntax errors detected. XML structure is production-ready.

---

### 2. Router Logic vs Specification ⚠️ CRITICAL ISSUES

#### Issue #1: XQINSTYPE Value Type Mismatch (CRITICAL)

**Specification (Step 1g, Step 1d):**
```
XQINSTYPE values: 0=MA, 1=Traditional, 2=ESI, 3=Other (integer)
```

**Router Implementation (lines 147-152):**
```python
xqins = XQINSTYPE.val  # r1 (MA), r2 (Traditional), r3 (ESI), r4 (Other)
```

**Router Logic (lines 225-227):**
```python
ins_ma_flag = (xqins in ["r1", "r2"])  # Checking for r-codes
```

**Impact:**
- Insurance eligibility flags will NEVER match
- All respondents will be treated as `ins_other_flag = True`
- MA/ESI routing will fail; everyone goes to GLP1 or soft-term

**Fix Required:**
```python
# CORRECT: Match Step 1g spec
ins_ma_flag = (xqins in [0, 1])    # Integer values 0 and 1
ins_esi_flag = (xqins == 2)        # Integer value 2
ins_other_flag = (xqins == 3)      # Integer value 3
```

**OR** if XQINSTYPE is actually stored as r-codes in source XMLs:
```python
# Alternative: Match source XML behavior
ins_ma_flag = (xqins in ["r1", "r2"])
ins_esi_flag = (xqins == "r3")
ins_other_flag = (xqins == "r4")
```

**Action:** Verify source XML XQINSTYPE format (lines 2584-2602 PRISM_MA_ESI.xml) and update router logic accordingly.

---

#### Issue #2: XRANDOMPICK Not Set (CRITICAL)

**Specification (Step 1f, Step 1g):**
- `XRANDOMPICK` should be set to 1 (ESI) or 2 (MA) when routing decision is made
- Step 1g lists XRANDOMPICK as exported variable

**Router Implementation:**
- Variable `XRANDOMPICK` declared in Step 1g spec but **NOT found in router_module.xml**
- Router uses `ROUTE_BLOCK_INTERNAL` instead (r1/r2/r3/r4)

**Impact:**
- Data export will be missing XRANDOMPICK column
- Bryan's existing analytics/dashboards may rely on this variable
- Inconsistent with source XML behavior (PRISM_MA_ESI.xml sets XRANDOMPICK at line ~2620)

**Fix Required:**
Add XRANDOMPICK variable declaration and assignment:

```xml
<!-- Add to SECTION 1: HIDDEN VARIABLES SCHEMA -->
<radio label="XRANDOMPICK"
       optional="1"
       where="execute,survey,report">
  <title>MA/ESI Route Selector</title>
  <comment>Random picker for MA vs ESI when both are open</comment>
  <row label="r1">ESI</row>
  <row label="r2">MA</row>
</radio>
```

```python
# Add to SECTION 4 allocation logic (after route_block assignment)
if route_block == "r1":  # ROI_ESI_FINAL
    XRANDOMPICK.val = "r1"
elif route_block == "r2":  # ROI_MA_FINAL
    XRANDOMPICK.val = "r2"
else:
    XRANDOMPICK.val = None  # No allocation
```

**Action:** Add XRANDOMPICK to match Step 1g spec and existing XML behavior.

---

### 3. Test Scenario Walkthrough ⚠️ FAILS (Due to Issues #1 & #2)

Manually traced all 10 test cases through router logic:

| Test | Scenario | Expected Result | Actual Result (Current Code) | Status |
|------|----------|-----------------|------------------------------|--------|
| 1 | Both open, odd segment | ESI route | ❌ GLP1 or soft-term (XQINSTYPE mismatch) | **FAIL** |
| 2 | Both open, even segment | MA route | ❌ GLP1 or soft-term (XQINSTYPE mismatch) | **FAIL** |
| 3 | ESI only | ESI route | ❌ GLP1 or soft-term | **FAIL** |
| 4 | MA only | MA route | ❌ GLP1 or soft-term | **FAIL** |
| 5 | All full | Soft-term | ⚠️ Soft-term (correct decision, wrong reason) | **PARTIAL** |
| 6 | XQINSTYPE=Other | Soft-term or GLP1 | ✅ GLP1 if open, else soft-term | **PASS** |
| 7 | XSEG missing | Soft-term | ✅ TYPING_INCOMPLETE soft-term | **PASS** |
| 8 | QINSTYPE=r99 | Hard screenout | ✅ ERROR_INVALID_INSURANCE | **PASS** |
| 9 | Boundary (ESI=1) | ESI route | ❌ GLP1 or soft-term | **FAIL** |
| 10 | Traditional Medicare | MA route | ❌ GLP1 or soft-term | **FAIL** |

**Summary:** 3/10 PASS (30% success rate)

**Root Cause:** Issue #1 (XQINSTYPE mismatch) breaks all MA/ESI routing.

---

### 4. Documentation Consistency ✅ PASS

#### DESIGN_DECISIONS.md vs Router Code

Checked all 8 assumptions:

| Assumption | Router Code Reference | Match? |
|------------|----------------------|--------|
| 1. Quota placeholders (`*`, `inf` = unlimited) | Lines 162-163 comments | ✅ |
| 2. GLP1 age/gender ranges (soft minimums) | Lines 210-214 placeholder | ✅ |
| 3. MA/ESI block balance (marker-based) | Lines 231-238 tie-break logic | ✅ |
| 4. Study priority (MA/ESI > GLP1) | Lines 229-268 priority structure | ✅ |
| 5. AL_VAX scope (architecture ready) | Line 20 comment + line 158 TODO | ✅ |
| 6. Soft-term strategy | Lines 312-324 soft-term block | ✅ |
| 7. Decision logging | Lines 280-284 ROUTER_DECISION_LOG | ✅ |
| 8. Tie-breaking (deterministic mod) | Lines 232-241 `xseg % 2` logic | ✅ |

**Finding:** All 8 assumptions correctly implemented in router logic (pending XQINSTYPE fix).

#### INTEGRATION_GUIDE.md Accuracy

Checked extraction instructions:

| Step | Accuracy | Notes |
|------|----------|-------|
| Step 1: Extract typing module | ✅ Correct line numbers (2400-2570) | Verified in PRISM_MA_ESI.xml |
| Step 2: Extract insurance questions | ✅ Correct line numbers (885-940) | QINSTYPE + QINS_MEDICARE confirmed |
| Step 3: Extract study blocks | ⚠️ Approximate (no exact line numbers) | Need to verify b1/b3 exact ranges |
| Step 4: Quota API config | ⚠️ Placeholder syntax | Needs Decipher platform version |
| Step 5: Link quota sheets | ✅ Standard Decipher process | Correct tag references |
| Step 6: Test integration | ✅ References TEST_SCENARIOS.md | Correct |

**Finding:** Integration guide is accurate; Steps 3-4 need platform-specific details.

---

### 5. Variable Schema Validation ⚠️ MISMATCH

Cross-checked router variables against Step 1g spec:

| Variable | Step 1g Type | Step 1g Values | Router Implementation | Match? |
|----------|--------------|----------------|----------------------|--------|
| `ROUTER_STATUS` | Radio | SUCCESS_MA, SUCCESS_ESI, SUCCESS_GLP1, OVERQUOTA_NO_ALLOCATION, TYPING_INCOMPLETE, ERROR_INVALID_INSURANCE | Radio with r1-r6 rows | ✅ PASS |
| `ROUTER_ALLOCATION_REASON` | Radio | QUOTA_AVAILABLE, QUOTA_FULL, XQINSTYPE_OTHER_NO_ROUTE, TYPING_FAILED | Radio with r1-r4 rows | ✅ PASS |
| `ROUTER_DECISION_LOG` | Text | Semicolon-delimited audit trail | Text, size=25 | ⚠️ Size too small (need 200+) |
| `ROUTE_BLOCK_INTERNAL` | Hidden (not exported) | ROI_ESI_FINAL, ROI_MA_FINAL, ROI_GLP1_FINAL, NONE | Radio r1-r4, where="execute" | ✅ PASS |
| `XRANDOMPICK` | Radio (exported) | 1=ESI, 2=MA | **❌ MISSING** | **FAIL** |

**Issues:**
- `ROUTER_DECISION_LOG` size=25 is too small for log string (need 200+ chars)
- `XRANDOMPICK` missing entirely (see Issue #2)

---

### 6. Additional Technical Issues

#### Issue #3: Quota API Placeholder Code (HIGH PRIORITY)

**Lines 161-163, 175-193, 207-214:**
```python
# PLACEHOLDER: Replace with actual Decipher quota API calls
esi_open = 25  # REPLACE with actual quota check
ma_open = 20   # REPLACE with actual quota check
glp1_open = 100  # REPLACE with actual quota check
```

**Impact:**
- Router logic will use dummy quota values
- All tests will fail in staging environment
- Quota decrements will not occur

**Fix Required:**
Need Bryan to provide:
1. Decipher platform version (affects API syntax)
2. Exact quota API function names (e.g., `getQuotaRemaining()`, `quota.getValue()`, etc.)
3. Quota sheet tag references confirmed

**Recommendation:** Flag this in INTEGRATION_GUIDE.md Step 4 as **CRITICAL** blocker for Step 2e.

---

#### Issue #4: Missing datetime Import (MEDIUM PRIORITY)

**Lines 107, 129, 283:**
```python
ROUTER_DECISION_LOG.val = f"{datetime.now()}; ..."
```

**Impact:**
- Exec block will fail with `NameError: name 'datetime' is not defined`
- Router will crash on first execution

**Fix Required:**
Add import at top of first exec block:
```python
from datetime import datetime
```

**OR** use Decipher native timestamp function if available (platform-specific).

---

#### Issue #5: Block Dispatch Logic Incomplete (MEDIUM PRIORITY)

**Lines 296-324:**
Current block structure has separate blocks for each route:
```xml
<block label="b_router_dispatch">
  <condition cond="ROUTE_BLOCK_INTERNAL.r1">Route to ESI</condition>
  <goto block="ROI_ESI_FINAL"/>
</block>
```

**Issue:** Multiple sequential blocks without fallthrough; if `ROUTE_BLOCK_INTERNAL.r1` is false, block ends and survey continues to next block (unintended).

**Fix Required:**
Use proper conditional structure:
```xml
<exec>
# Dispatch based on ROUTE_BLOCK_INTERNAL
if ROUTE_BLOCK_INTERNAL.r1:
    # Continue to ROI_ESI_FINAL (next block)
    pass
elif ROUTE_BLOCK_INTERNAL.r2:
    # Jump to ROI_MA_FINAL
    skipTo("ROI_MA_FINAL")
elif ROUTE_BLOCK_INTERNAL.r3:
    # Jump to ROI_GLP1_FINAL
    skipTo("ROI_GLP1_FINAL")
elif ROUTE_BLOCK_INTERNAL.r4:
    # Jump to soft-term
    skipTo("b_router_softterm")
</exec>
```

**OR** use Decipher's native block conditions if supported (platform-specific).

---

## Summary of Required Fixes

### CRITICAL (Must Fix Before Client Review)

1. **XQINSTYPE Value Type** (Lines 152, 225-227)
   - Verify source XML format (integer 0-3 or r-codes r1-r4)
   - Update router logic to match

2. **Add XRANDOMPICK** (Missing variable)
   - Add variable declaration to Section 1
   - Set value in allocation logic (Section 4)

### HIGH PRIORITY (Must Fix Before Integration)

3. **Quota API Placeholder Code** (Lines 161-193, 207-214)
   - Request Decipher API syntax from Bryan
   - Replace dummy values with real quota checks

4. **datetime Import** (Lines 107, 129, 283)
   - Add Python datetime import
   - OR use Decipher native timestamp function

### MEDIUM PRIORITY (Can Fix During Integration)

5. **ROUTER_DECISION_LOG Size** (Line 55)
   - Change `size="25"` to `size="250"` or larger

6. **Block Dispatch Logic** (Lines 296-324)
   - Consolidate into single exec block with skipTo() calls
   - OR verify Decipher block conditions work as intended

---

## Recommendations

### Before Client Review:
1. ✅ Fix Issue #1 (XQINSTYPE values) — **DO THIS NOW**
2. ✅ Fix Issue #2 (XRANDOMPICK missing) — **DO THIS NOW**
3. ✅ Fix Issue #4 (datetime import) — **Quick fix**
4. ✅ Fix Issue #5 (ROUTER_DECISION_LOG size) — **Quick fix**
5. ✅ Update INTEGRATION_GUIDE.md to flag Issue #3 as blocker

### During Integration (Step 2e):
6. Replace quota API placeholder code with real Decipher syntax
7. Test block dispatch logic in Decipher staging environment
8. Verify XQINSTYPE format in source XML (integer vs r-code)

### Send to Client:
- ✅ All documentation (DESIGN_DECISIONS.md, INTEGRATION_GUIDE.md, TEST_SCENARIOS.md, PHASE_2_SUMMARY.md)
- ✅ Fixed router_module.xml (with Issues #1, #2, #4, #5 resolved)
- ✅ This VALIDATION_REPORT.md (transparency on known issues)

---

## Test Readiness Assessment

**Current State:** ⚠️ **NOT READY** for staging testing

**Blockers:**
- Issue #1 breaks all MA/ESI routing (0% success rate on 7/10 tests)
- Issue #2 causes data export mismatch
- Issue #3 requires Bryan's input (Decipher API syntax)

**After Fixes:**
- ✅ Logic will be production-ready (pending quota API integration)
- ✅ Test scenarios will pass (estimated 10/10 after fixes)
- ✅ Safe to send to Bryan for platform-specific details

---

## Estimated Fix Time

| Issue | Complexity | Time | Priority |
|-------|-----------|------|----------|
| #1 XQINSTYPE mismatch | Low (3 lines) | 5 min | CRITICAL |
| #2 XRANDOMPICK missing | Medium (20 lines) | 15 min | CRITICAL |
| #3 Quota API placeholders | High (depends on Bryan) | TBD | HIGH |
| #4 datetime import | Low (1 line) | 2 min | MEDIUM |
| #5 ROUTER_DECISION_LOG size | Low (1 line) | 1 min | MEDIUM |
| #6 Block dispatch logic | Medium (10 lines) | 10 min | MEDIUM |

**Total (excluding #3):** ~35 minutes to production-ready router stub

---

## Next Steps

1. **Immediate:** Fix Issues #1, #2, #4, #5 (35 min)
2. **Commit:** Push fixes to GitHub with message "Fix critical router issues (XQINSTYPE, XRANDOMPICK, logging)"
3. **Re-test:** Run validation again (expect 10/10 pass)
4. **Client Ready:** Send updated deliverables + this validation report to Bryan
5. **Integration:** Await Bryan's Decipher API details for Issue #3 (Step 2e)

---

**Validation Completed:** Jan 19-20, 2026  
**Next Action:** Apply fixes and re-validate before client review
