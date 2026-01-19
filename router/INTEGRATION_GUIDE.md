# Router Integration Guide

**Target:** Integrate `router_module.xml` into existing PRISM survey XMLs  
**Date:** Jan 19, 2026  
**Audience:** Survey programmers, Bryan's team

---

## Integration Overview

The router module is designed as a **single-survey shell** that routes respondents to the appropriate study block based on their segment, insurance type, and open quotas. This guide shows how to integrate it with existing PRISM XMLs.

---

## Prerequisites

**Required files:**
- `router_module.xml` (this router)
- `PRISM_MA_ESI.xml` (source XML with typing + MA/ESI blocks)
- `PRISM_GLP1` (source XML with GLP1 study content)
- `MA_ESI_quota.xls` (Decipher quota sheets)
- `GLP1-quota.xls` (Decipher quota sheets)

**Optional:**
- `PRISM_AL_VAX` (if AL_VAX study in scope)
- AL_VAX quota sheet (request from Bryan if needed)

---

## Integration Steps

### Step 1: Extract Typing Module from Source XMLs

**Source:** `PRISM_MA_ESI.xml` lines ~2400-2570

**What to extract:**
- GOP scoring exec block (`algorithmRaw`, `algorithmCalculation`)
- DEM scoring exec block
- `XSEG_ASSIGNED` assignment logic
- `XGOP_SEG_FINAL_1/2`, `XDEM_SEG_FINAL_1/2` variables

**Where to place in router:**
- **Before** Section 2 (Typing Module Integration) in `router_module.xml`

**Example:**
```xml
<!-- EXTRACTED FROM PRISM_MA_ESI.xml lines 2400-2570 -->
<exec>
# GOP scoring logic
# (paste GOP algorithmRaw + algorithmCalculation here)
</exec>

<exec>
# DEM scoring logic
# (paste DEM algorithmRaw + algorithmCalculation here)
</exec>

<exec>
# XSEG_ASSIGNED assignment
if XGOP_SEG_FINAL_1.val:
    XSEG_ASSIGNED.val = XGOP_SEG_FINAL_1.val
elif XDEM_SEG_FINAL_1.val:
    XSEG_ASSIGNED.val = XDEM_SEG_FINAL_1.val + 10
</exec>
```

---

### Step 2: Extract Insurance Questions

**Source:** `PRISM_MA_ESI.xml` lines 885-940

**What to extract:**
- `QINSTYPE` question (radio, r1-r4, r7, r99)
- `QINS_MEDICARE` question (radio, r1-r2, conditional on QINSTYPE.r7)
- `term_QINSTYPE` termination logic (if r99 selected)
- `XQINSTYPE` derivation exec

**Where to place in router:**
- **Before** Section 3 (Insurance Classification) in `router_module.xml`

**Example:**
```xml
<!-- EXTRACTED FROM PRISM_MA_ESI.xml lines 885-940 -->
<radio label="QINSTYPE">
  <title>What is the main type of health insurance coverage that you currently have?</title>
  <row label="r1">A plan through my employer</row>
  <row label="r2">A plan through my spouse's employer</row>
  <row label="r3">A plan through my parent's employer</row>
  <row label="r4">A plan you purchased yourself directly from an insurance company</row>
  <row label="r7">Medicare</row>
  <row label="r99">Prefer not to answer</row>
</radio>

<radio label="QINS_MEDICARE" cond="QINSTYPE.r7">
  <title>Do you have traditional or original Medicare, or do you have a Medicare Advantage plan?</title>
  <row label="r1">Traditional Medicare</row>
  <row label="r2">Medicare Advantage</row>
</radio>

<exec>
# XQINSTYPE derivation
if QINS_MEDICARE.r2:
    XQINSTYPE.val = "r1"  # Medicare Advantage
elif QINS_MEDICARE.r1:
    XQINSTYPE.val = "r2"  # Traditional Medicare
elif QINSTYPE.r1 or QINSTYPE.r2 or QINSTYPE.r3:
    XQINSTYPE.val = "r3"  # Employer-Sponsored Insurance (ESI)
else:
    XQINSTYPE.val = "r4"  # Other
</exec>
```

---

### Step 3: Extract Study Content Blocks

**For ESI block:**
- Source: `PRISM_MA_ESI.xml` lines ~2700+ (block `b1` or `ROI_ESI_FINAL`)
- Copy all ESI-specific questions
- Paste into `router_module.xml` Section 6, block `ROI_ESI_FINAL`

**For MA block:**
- Source: `PRISM_MA_ESI.xml` lines ~2800+ (block `b3` or `ROI_MA_FINAL`)
- Copy all MA-specific questions
- Paste into `router_module.xml` Section 6, block `ROI_MA_FINAL`

**For GLP1 block:**
- Source: `PRISM_GLP1` (entire study content after typing)
- Copy all GLP1 questions
- Paste into `router_module.xml` Section 6, block `ROI_GLP1_FINAL`

---

### Step 4: Configure Quota Sheet References

**In Section 4 (Quota Allocator), replace placeholder logic with actual Decipher quota API calls:**

**Example (Decipher syntax):**
```xml
<exec>
# Read Block_INSTYPE_Quota
esi_block_remaining = getQuotaCell("Block_INSTYPE_Quota", f"XQINSTYPE_{xqins}", "ESI_SEC").remaining
ma_block_remaining = getQuotaCell("Block_INSTYPE_Quota", f"XQINSTYPE_{xqins}", "MA_SEC").remaining

# Read Wave One Equal Allocation
esi_wave_remaining = getQuotaCell("Wave One Equal Allocation QUOTA", f"Segment_{xseg}", "ESI_SEC").remaining
ma_wave_remaining = getQuotaCell("Wave One Equal Allocation QUOTA", f"Segment_{xseg}", "MA_SEC").remaining

# Compute availability
esi_open = min(esi_block_remaining, esi_wave_remaining)
ma_open = min(ma_block_remaining, ma_wave_remaining)

# GLP1 quota
glp1_segment_remaining = getQuotaCell("Wave One Equal Allocation QUOTA", f"Segment_{xseg}", "GLP1").remaining
glp1_open = glp1_segment_remaining  # Simplified; add age/gender checks if needed
</exec>
```

**Note:** Exact Decipher API syntax depends on platform version. Consult Decipher documentation or existing XML examples.

---

### Step 5: Link Quota Sheets in Survey Settings

**In Decipher platform:**
1. Upload `MA_ESI_quota.xls` and `GLP1-quota.xls` to survey
2. Configure quota tags:
   - `quo_INSTYPE` → Block_INSTYPE_Quota sheet
   - `quo13` → Wave One Equal Allocation QUOTA sheet
   - `quo1`, `quo2`, `DemQuota` → other sheets as needed
3. Set `overquota="noqual"` behavior (existing default)

---

### Step 6: Test Integration

**Use test scenarios from `router/tests/TEST_SCENARIOS.md` (see Step 6 below):**

1. **Test Case 1:** Both MA & ESI open, odd segment → expect SUCCESS_ESI
2. **Test Case 2:** Both MA & ESI open, even segment → expect SUCCESS_MA
3. **Test Case 5:** All quotas full → expect OVERQUOTA_NO_ALLOCATION + soft-term message
4. **Test Case 7:** Typing incomplete → expect TYPING_INCOMPLETE error
5. **Test Case 10:** Traditional Medicare → expect SUCCESS_MA

**Validation:**
- Check `ROUTER_DECISION_LOG` export for all test respondents
- Verify no hard terminations on overquota (soft-term only)
- Confirm quota decrements correctly in Decipher sheets

---

## Integration Checklist

- [ ] Extract typing module from PRISM_MA_ESI.xml → paste into router
- [ ] Extract insurance questions (QINSTYPE, QINS_MEDICARE, XQINSTYPE) → paste into router
- [ ] Extract ESI study block → paste into `ROI_ESI_FINAL`
- [ ] Extract MA study block → paste into `ROI_MA_FINAL`
- [ ] Extract GLP1 study content → paste into `ROI_GLP1_FINAL`
- [ ] Replace quota placeholder logic with actual Decipher API calls
- [ ] Upload quota sheets to Decipher platform
- [ ] Configure quota tag mappings in survey settings
- [ ] Test all 10 scenarios from `router/tests/TEST_SCENARIOS.md`
- [ ] Validate `ROUTER_DECISION_LOG` exports correctly
- [ ] Verify soft-term behavior (no hard termination on overquota)
- [ ] Deploy to staging environment
- [ ] Client review & approval (Bryan)
- [ ] Deploy to production (go-live Jan 24)

---

## Troubleshooting

### Issue: Typing incomplete (ROUTER_STATUS = TYPING_INCOMPLETE)
**Cause:** `XSEG_ASSIGNED` not set before router logic runs  
**Fix:** Ensure typing exec blocks run BEFORE Section 2 in router XML

### Issue: Quota checks return incorrect values
**Cause:** Quota sheet not linked or tag misconfigured  
**Fix:** Verify quota sheet upload + tag mappings in Decipher platform

### Issue: Respondents hard-terminated on overquota (not soft-term)
**Cause:** `<term>` node present in overquota path  
**Fix:** Remove `<term>` nodes from router; use soft-term message block only

### Issue: ROUTER_DECISION_LOG not exported
**Cause:** Variable not set to `where="execute,survey,report"`  
**Fix:** Verify `ROUTER_DECISION_LOG` has correct `where` attribute (line 39 in router_module.xml)

---

## Next Steps

1. Complete integration following checklist above
2. Run all test scenarios (see `router/tests/TEST_SCENARIOS.md`)
3. Deploy to staging for Bryan's review
4. Address any feedback/adjustments
5. Go-live to production (target: Jan 24)

---

## Contact

For questions or issues during integration:
- Review `router/DESIGN_DECISIONS.md` for assumptions/rationale
- Check `discovery/` folder for detailed system documentation
- Consult Bryan for clarifications on quota logic or study priority
