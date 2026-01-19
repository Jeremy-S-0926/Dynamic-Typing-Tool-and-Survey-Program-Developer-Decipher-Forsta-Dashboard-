# PROJECT STATUS & TRACKING

**Project:** PRISM Dynamic Typing Tool & Router  
**Client:** Bryan Dumont, Reservoir Communications  
**Status:** Week 1 - Step 1 Discovery COMPLETE ✅ (All 8 steps: 1a-1h)  
**Date:** Jan 20, 2026  
**Hours Logged:** ~25 / 90-130 estimated

---

## Current Phase: Step 1 - Discovery & Analysis (13-22 hours total)

### Completed Tasks ✅

#### Step 1a: Review Existing XML + Quota Structure (✅ Jan 19)
- Reviewed PRISM_MA_ESI.xml, PRISM_GLP1, PRISM_AL_VAX
- Identified `XSEG_ASSIGNED` as the live segment output (16 segments, stable across studies)
- Confirmed quotas are read live from Decipher sheets (no hard-coded caps)
- Documented current routing logic (MA/ESI via XQINSTYPE + XRANDOMPICK)
- **Deliverable:** `discovery/PRISM_System_Baseline.md`

#### Step 1b: Map Segment Definitions & Typing Algorithm (✅ Jan 19)
- Listed all 16 segment names (stable across MA/ESI, GLP1, AL_VAX)
- Documented GOP (10 segments) + DEM (6 segments) scoring logic
- Confirmed scoring uses z-scores + unweighted squared-distance to centroids
- Verified tie-breaking: deterministic sort on distance values
- **Deliverable:** `discovery/Step_1b_Segment_Map.md`

#### Step 1c: Parse Quota Sheets (✅ Jan 19)
- Extracted MA_ESI_quota.xls: 25 sheets with quota definitions
  - Block_INSTYPE_Quota: XQINSTYPE rows with ESI_SEC/MA_SEC caps (75/200, 75/200, 750/200, 300/200)
  - Wave One Equal Allocation: segment caps (ESI=75, MA=50 per segment)
  - All other quota rows: `inf` (open)
- Extracted GLP1_quota.xls: 19 sheets
  - Wave One Equal Allocation: numeric segment caps (range 30-350)
  - Age Quota: ranges (e.g., 18-29: 250-450)
  - Gender Quota: ranges (Male/Female: 1100-1500; Other 0-100)
  - All other quota rows: `inf`
- **Deliverable:** `discovery/Quota_Map.md` + `discovery/Step_1c_Quota_Status.md`
- **Open Questions:**
  - What do `*` (Total Quota) and `inf` placeholders mean? (no cap vs. open)
  - GLP1 age/gender ranges: are both bounds enforced or only upper?
  - MA/ESI Block c1/c2 are uncapped; should they be set for deterministic 50/50?
  - Need AL_VAX and other study quota sheets if in scope

#### Step 1d: Map Insurance Classification Logic (✅ Jan 19)
- Reviewed PRISM_MA_ESI.xml insurance questions:
  - QINSTYPE (main type: employer, spouse, parent, self, Medicare)
  - QINS_MEDICARE (shows if Medicare: Traditional or Advantage)
  - Derived XQINSTYPE: 0=MA, 1=Traditional, 2=ESI, 3=Other
  - Derived XRANDOMPICK: MA/ESI selector when both quotas open
- Verified identical logic in PRISM_GLP1 (has XRANDOMPICK, no QINSTYPE/QINS_MEDICARE)
- Insurance eligibility flags for router:
  - `INS_MA_FLAG` = XQINSTYPE ∈ {0,1}
  - `INS_ESI_FLAG` = XQINSTYPE == 2
  - `INS_OTHER_FLAG` = XQINSTYPE == 3
- **Deliverable:** `discovery/Step_1d_Insurance_Logic.md`

#### Step 1e: Identify Termination/Redirect Codes + Panel Constraints (✅ Jan 20)
- Found hard termination codes: `term_QINSTYPE` (r99 screenout), `Term_QS1`, `Term_QS2` (statement qual), `Term_PARTYID`, `Term_VOTE24`, `Term_QPE2`, `Term_QPE3`
- Documented current overquota behavior: hard-term with Dynata status codes (rst=1/2/3)
- Identified panel constraints: Dynata 24-hr re-entry block on hard-term, status field limits (0–9), session time limits (5 min typing + 30 sec allocator)
- Designed soft-term strategy: no status code written, ROUTER_STATUS tagged for recovery, respondent remains eligible if quotas reopen
- **Deliverable:** `discovery/Step_1e_Termination_Redirect.md`

#### Step 1f: Design Router Decision Tree (✅ Jan 20)
- Wrote complete router pseudocode (typing → insurance → quota allocator → route/soft-term)
- Defined tie-breaking: deterministic mod-based (odd segments → ESI, even → MA) for reproducibility
- Created flowchart (ASCII) showing all decision paths (success, overquota, error, incomplete)
- Documented quota read timing: single read at entry, live from Decipher sheets
- **Deliverable:** `discovery/Step_1f_Router_Logic.md`

#### Step 1g: Build Output Schema & Hidden Variables (✅ Jan 20)
- Designed hidden variables: XSEG_ASSIGNED (typing), XQINSTYPE (insurance), XRANDOMPICK (allocator), ROUTER_STATUS (main decision), ROUTER_ALLOCATION_REASON (secondary), ROUTER_DECISION_LOG (audit trail)
- Created Decipher XML implementation guide (hidden="true", export="true" declarations)
- Defined data export schema (analyst export: XSEG, XQINS, XRANDOMPICK, ROUTER_STATUS, ROUTER_DECISION_LOG; QA debug: XGOP_RAW, XDEM_RAW, QUOTA_STATE)
- Documented ROUTER_DECISION_LOG format (semicolon-delimited, max 500 chars, includes timestamp, segment, insurance, quota states, decision, status)
- **Deliverable:** `discovery/Step_1g_Output_Schema.md`

#### Step 1h: Build Test Scenarios (✅ Jan 20)
- Created 10 test cases covering all router paths:
  - Test 1: Both open, odd segment (tie-break ESI)
  - Test 2: Both open, even segment (tie-break MA)
  - Test 3: ESI only (MA full)
  - Test 4: MA only (ESI full)
  - Test 5: Both full (overquota soft-term)
  - Test 6: XQINSTYPE=Other (ineligible type, soft-term)
  - Test 7: XSEG_ASSIGNED missing (typing incomplete)
  - Test 8: QINSTYPE=r99 (hard screenout, preserved existing logic)
  - Test 9: Marginal quota boundary (1 slot left)
  - Test 10: Traditional Medicare (XQINSTYPE=1 eligible for MA)
- Validated consistency checks (tie-break rule, overquota status, audit logs, soft vs hard-term)
- **Deliverable:** `discovery/Step_1h_Test_Scenarios.md`

---

### Pending Tasks 🚧

**NONE - Step 1 Complete!**

Step 1 (Discovery & Analysis, 13-22 hours target) now **COMPLETE** as of Jan 20, 2026.

---

### Timeline Summary (Step 1)

| Task | Duration | Status | Deliverable |
|------|----------|--------|-------------|
| 1a: Review XMLs | 4–6 hrs | ✅ Jan 19 | PRISM_System_Baseline.md |
| 1b: Map segments | 3–4 hrs | ✅ Jan 19 | Step_1b_Segment_Map.md |
| 1c: Parse quotas | 2–3 hrs | ✅ Jan 19 | Quota_Map.md + Step_1c_Quota_Status.md |
| 1d: Insurance logic | 2–3 hrs | ✅ Jan 19 | Step_1d_Insurance_Logic.md |
| 1e: Termination/redirects | 2–3 hrs | ✅ Jan 20 | Step_1e_Termination_Redirect.md |
| 1f: Router pseudocode | 3–4 hrs | ✅ Jan 20 | Step_1f_Router_Logic.md |
| 1g: Logging schema | 2–3 hrs | ✅ Jan 20 | Step_1g_Output_Schema.md |
| 1h: Test scenarios | 2–3 hrs | ✅ Jan 20 | Step_1h_Test_Scenarios.md |
| **Total** | **13–22 hrs** | **~20 hrs logged** | **8 docs complete** |

---

## Workspace Organization

```
.
├── README.md                      # Project overview
├── PROJECT_STATUS.md              # This file (tracking + planning)
├── chats/                         # Client communications (consolidated)
│   ├── 2026-01-12_Initial_Scope.md
│   ├── 2026-01-17_Problem_Statement.md
│   └── 2026-01-18_Status_Check.md
├── source/                        # Original files (do not edit)
│   ├── PRISM_MA_ESI.xml
│   ├── PRISM_GLP1
│   ├── PRISM_AL_VAX
│   ├── PRISM_Segmentation_Typing_Tools.xlsx
│   ├── MA_ESI_quota.xls
│   └── GLP1-quota.xls
├── discovery/                     # Analysis & mapping docs
│   ├── PRISM_System_Baseline.md           # ✅ Step 1a
│   ├── Step_1b_Segment_Map.md             # ✅ Step 1b
│   ├── Step_1c_Quota_Status.md            # ✅ Step 1c
│   ├── Quota_Map.md                       # ✅ Step 1c
│   ├── Step_1d_Insurance_Logic.md         # ✅ Step 1d
│   ├── Step_1e_Termination_Redirect.md    # ✅ Step 1e
│   ├── Step_1f_Router_Logic.md            # ✅ Step 1f
│   ├── Step_1g_Output_Schema.md           # ✅ Step 1g
│   └── Step_1h_Test_Scenarios.md          # ✅ Step 1h
├── router/                        # Router implementation (Phase 2+)
│   ├── router_module_stub.xml            # (not started)
│   ├── integration_tests.md               # (not started)
│   └── deployment_checklist.md            # (not started)
└── .gitignore
```

---

## Key Findings Summary

### Segment System
- **16 stable segments** used across all studies
- GOP scoring: 10 segments (Consumer Empowerment Champions → Trust The Science Pragmatists)
- DEM scoring: 6 segments (Idealists → Institutionalists)
- Output: `XSEG_ASSIGNED` (values 1–16)
- Typing completion: guaranteed (no term nodes between scoring and assignment)

### Insurance Routing (MA/ESI variant)
- QINSTYPE + QINS_MEDICARE → XQINSTYPE (4 classes)
- XQINSTYPE determines eligibility for MA vs ESI
- When both quotas open: XRANDOMPICK picks one (marker-based or 50/50)
- Block_INSTYPE_Quota caps by XQINSTYPE row (ESI_SEC / MA_SEC columns)

### Quotas
- **MA/ESI:** Block-level caps (ESI 75–750, MA 200 per XQINSTYPE) + segment caps (ESI 75, MA 50)
- **GLP1:** Segment-level numeric caps (30–350), age/gender ranges
- All sheets read live from Decipher (no local counts)
- Overquota default: soft-term + `ROUTER_STATUS=OVERQUOTA_TAGGED`

### Open Questions for Bryan
1. Interpretation of `*` (Total) and `inf` placeholders in quota sheets
2. GLP1 age/gender min-max ranges: both bounds enforced or just upper?
3. MA/ESI Block c1/c2 balance: should caps be set or remain dynamic?
4. Provide AL_VAX and other study quota sheets if in scope
5. Study priority order: MA > ESI > GLP1 > others (confirm or adjust)

---

## Next Actions

**Immediate (complete):**
- ✅ Step 1a–1h all documented
- ✅ 8 discovery documents created
- ✅ Workspace cleaned (10 redundant files removed)
- ✅ Ready for Phase 2 (XML stub implementation)

**Week 1 (by Jan 24):**
1. Clarifications from Bryan (quota placeholders, study priority, AL_VAX sheets)
2. Phase 2: Create router XML stub (minimal MVP for live deployment)
   - Incorporate Steps 1e–1h logic
   - Test with mock Dynata panel
   - Validate soft-term behavior

**Week 2 (by Jan 31):**
1. Deploy router to staging
2. Field validation + QA
3. Go-live (target: early Feb)

---

## Hours Summary

| Phase | Task | Hours | Status | Date |
|-------|------|-------|--------|------|
| Step 1 | 1a: Review XMLs | 5 | ✅ | Jan 19 |
| Step 1 | 1b: Map segments | 3.5 | ✅ | Jan 19 |
| Step 1 | 1c: Parse quotas | 2.5 | ✅ | Jan 19 |
| Step 1 | 1d: Insurance logic | 2.5 | ✅ | Jan 19 |
| Admin | Workspace cleanup | 0.5 | ✅ | Jan 19 |
| Step 1 | 1e: Termination/redirects | 2.5 | ✅ | Jan 20 |
| Step 1 | 1f: Router pseudocode | 3.5 | ✅ | Jan 20 |
| Step 1 | 1g: Output schema | 2.5 | ✅ | Jan 20 |
| Step 1 | 1h: Test scenarios | 2.5 | ✅ | Jan 20 |
| **Total** | **Step 1 + Admin** | **~25 hrs** | **Complete** | **Jan 20** |

*Target was 13–22 hours; complexity of quota system + 4 additional discovery docs added ~3 hours. All work within scope.*

---

## Contact Notes

- **Client:** Bryan Dumont, Reservoir Communications
- **Urgency:** Live fielding resumes next week (4 concurrent studies)
- **Scope:** MVP router only; full architecture hardening to follow
- **Format:** Prefer text over chat/calls (faster, documented)
