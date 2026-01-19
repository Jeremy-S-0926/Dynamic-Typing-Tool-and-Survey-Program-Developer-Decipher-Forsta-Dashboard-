Created 12 todos

## Step 1: Discovery & Analysis Phase

### **Focus: Understand the Existing System (13-22 hours total)**

This is the foundation layer. Without this, the router will break on live sample.

---

### **Step 1a: Review Existing [[PRISM XML + Quota Structure]]**
**Timeline:** 4-6 hours  
**Deadline:** Jan 19 evening

**Tasks:**
1. Pull and open all three gists:
   - [[`PRISM_MA_ESI.xml` (2,500 lines - typing + MA/ESI branching)]]
   - [[`PRISM_GLP1` (2,000 lines - GLP-1 variant)]]
   - [[`PRISM_AL_VAX` (2,200 lines - vaccine variant)]]

2. Find and document:
   - **Typing output variable name:** Is it `XSEG_ASSIGNED`? Or something else like `SEG_PRIMARY`, `SEGMENT`, etc?
   - **Where typing completes:** Search for `<exec>` blocks that compute segment; look for termination before assignment
   - **Existing quota variables:** Names like `MA_QUOTA`, `QUOTA_SEG1`, etc.
   - **Current quota check logic:** How does survey know if MA is full? Hard-coded thresholds or sheet read?
   - **Insurance flags:** Any existing `INS_MA_FLAG`, `INS_ESI_FLAG`? Or computed on the fly?
   - **Termination codes:** Search for `<term>`, redirect URLs, status codes
   - **Study branching:** How does survey route to MA vs ESI today?

3. Create reference document: **PRISM_System_Baseline.md** with:
   - Variable map (what each key variable contains)
   - Typing algorithm summary (1 paragraph)
   - Current quota structure
   - Current branching logic (diagram ASCII if helpful)

**Deliverable:** Baseline understanding of what's already built

**Questions for Bryan (if unclear):**
- Is `XSEG_ASSIGNED` the correct variable name for segment output?
- Are quotas currently in Decipher sheets or hard-coded in XML?
- What termination code should overquota respondents get?

---

### **Step 1b: Map Segment Definitions and Typing Algorithm**
**Timeline:** 3-4 hours  
**Deadline:** Jan 19 night / Jan 20 morning

**Tasks:**
1. Extract from PRISM XMLs:
   - **Segment names:** List all ~16 segment IDs (e.g., "Progressive", "Conservative", etc.)
   - **Scoring logic:** Which survey items/questions feed into segment scoring?
   - **Weights:** Are items weighted equally or scaled?
   - **Thresholds:** How are scores converted to segment assignment?
   - **Tie-breaking:** What happens if two segments are equally likely?

2. Create **Segment_Definition_Reference.xlsx** with columns:
   - Segment ID
   - Segment Name
   - Input Questions
   - Scoring Rule (pseudo-code)
   - Example Respondent Path

3. Verify segment stability claim:
   - Are the same 16 segments used in all three study XMLs?
   - Any segment-definition tweaks by study? Document if yes.

**Deliverable:** Clean segment reference for router logic building

---

### **Step 1c: Collect and Parse Current Quota Sheet**
**Timeline:** 2-3 hours  
**Deadline:** Jan 20 morning

**Tasks:**
1. Obtain `PRISM Segmentation Typing Tools.xlsx` from Bryan (or from gist if attached):
   - Open quota tabs
   - Document structure:
     - Rows: Segment names
     - Columns: Study names (MA, ESI, GLP1, PULSE_AL, PULSE_VAX, etc.)
     - Values: Current cap / current count

2. Map to Decipher quota sheet reference:
   - Sheet name Decipher reads from: (e.g., `sheet="Total Quota"`)
   - Row/column labels Decipher expects
   - Live update frequency (manual adjustment interval?)

3. Create **Quota_Map.md**:
   - Table of segment × study quotas
   - Current open/closed status
   - Priority order: MA > ESI > GLP1 > PULSE_AL > PULSE_VAX (or confirm actual order)

**Deliverable:** Live quota read points identified

---

### **Step 1d: Map Insurance Classification Logic**
**Timeline:** 2-3 hours  
**Deadline:** Jan 20 morning

**Tasks:**
1. Search existing XMLs for insurance-related questions:
   - Medicare Advantage coverage question (INS_MA_FLAG trigger)
   - Employer-sponsored insurance question (INS_ESI_FLAG trigger)
   - Other insurance types (INS_OTHER_FLAG)

2. Document exact conditions:
   - Question ID (e.g., QINS_TYPE)
   - Response options that set INS_MA=1
   - Response options that set INS_ESI=1
   - If both apply (unlikely but possible?): precedence

3. Check: Are INS_* flags already calculated in XMLs?
   - If yes: copy existing logic
   - If no: design new questions or piggyback on existing ones

**Deliverable:** Insurance flag logic ready to drop into router

---

### **Step 1e: Identify Termination/Redirect Codes + Panel Constraints**
**Timeline:** 2-3 hours  
**Deadline:** Jan 20 afternoon

**Tasks:**
1. Find in XMLs:
   - Hard termination code (e.g., `<term label="Term_QS1" cond="QS1.r2">Screened out</term>`)
   - Current overquota termination code (if exists): what message, what status sent to panel?
   - Redirect URLs or entry/exit logic
   - Panel vendor constraints (re-entry? time-out? status field limits?)

2. Design soft-term for overquota:
   - Polite message: "Thank you for your time. Unfortunately our quotas have been filled..."
   - Termination code: `TERM_OVERQUOTA_TAGGED` or similar
   - Status to panel: `status=overquota_tagged&seg=` + segment value

3. Verify reversibility:
   - Can respondents be recovered if quotas re-open? (design for yes)
   - Are there any panel vendor hard-constraints? (document)

**Deliverable:** Safe, reversible overquota exit plan

---

### **Step 1f: Design Router Decision Tree (Pseudocode)**
**Timeline:** 3-4 hours  
**Deadline:** Jan 20 afternoon

**Tasks:**
1. Write pseudocode for entire router flow:

```
ROUTER DECISION TREE (Pseudocode)

IF XSEG_ASSIGNED is empty:
  ERROR: Typing failed. Terminate.

// Insurance classification
IF [insurance Q] = MA:
  INS_MA_FLAG = 1
  STUDY_CANDIDATES = [MA, ESI, GLP1, others]
ELSE IF [insurance Q] = ESI:
  INS_ESI_FLAG = 1
  STUDY_CANDIDATES = [ESI, MA, GLP1, others]
ELSE:
  INS_MA_FLAG = 0, INS_ESI_FLAG = 0
  STUDY_CANDIDATES = [GLP1, PULSE_AL, PULSE_VAX, others]

// Build eligibility flags
ELIG_MA = INS_MA_FLAG AND [MA-specific criteria]
ELIG_ESI = INS_ESI_FLAG AND [ESI-specific criteria]
ELIG_GLP1 = [GLP1 criteria]
... etc for all studies

// Router decision
STUDY_ASSIGNED = NULL
STUDY_INTENT = NULL

FOR study IN [MA, ESI, GLP1, PULSE_AL, PULSE_VAX]:
  IF ELIG_[study] = 1:
    study_quota_remaining = READ_QUOTA(sheet, XSEG_ASSIGNED, study)
    
    IF study_quota_remaining > 0:
      STUDY_ASSIGNED = study
      STUDY_INTENT = study
      ROUTER_STATUS = "ASSIGNED"
      BREAK
    ELSE:
      STUDY_INTENT = study  // Remember first eligible
      CONTINUE

IF STUDY_ASSIGNED is NULL:
  // All quotas full or no eligibility
  ROUTER_STATUS = "OVERQUOTA_TAGGED"
  STUDY_INTENT = [first eligible or highest priority]
  LOG: (XSEG_ASSIGNED, eligibility_flags, quotas_full)
  SOFT_TERM with friendly message
ELSE:
  // Route to study block
  ROUTE_TO_[study] = 1
  LOG: success

OUTPUT: XSEG_ASSIGNED, STUDY_ASSIGNED, STUDY_INTENT, ROUTER_STATUS, decision_path
```

2. Define tie-breaking (if two studies equally viable):
   - Priority weight: MA > ESI > others (or confirm order with Bryan)
   - Deterministic secondary: `respondent_id % num_candidates`

3. Create **Router_Logic.md** with this pseudocode + flowchart diagram

**Deliverable:** Clear decision logic before coding

---

### **Step 1g: Build Logging/Hidden Variable Schema**
**Timeline:** 2-3 hours  
**Deadline:** Jan 20 evening

**Tasks:**
1. Design hidden variables (survey won't display but will export):

| Variable | Type | Example | Purpose |
|----------|------|---------|---------|
| `XSEG_ASSIGNED` | string | "Progressive" | Segment output |
| `STUDY_ASSIGNED` | string | "MA" or "OVERQUOTA_TAGGED" | Final routing decision |
| `STUDY_INTENT` | string | "MA" | Which study would have taken them |
| `ROUTER_STATUS` | string | "ASSIGNED" \| "OVERQUOTA_TAGGED" | Outcome |
| `ELIG_MA` | binary | 0 \| 1 | Eligibility flags |
| `ELIG_ESI` | binary | 0 \| 1 | Eligibility flags |
| `ELIG_GLP1` | binary | 0 \| 1 | Eligibility flags |
| `INS_MA_FLAG` | binary | 0 \| 1 | Insurance type |
| `INS_ESI_FLAG` | binary | 0 \| 1 | Insurance type |
| `QUOTAS_MA_SEG_REMAINING` | integer | 45 | QA diagnostics |
| `QUOTAS_ESI_SEG_REMAINING` | integer | 0 | QA diagnostics |
| `ROUTER_DECISION_PATH` | text | "MA eligible, quota full; ESI eligible, quota open → ASSIGNED=ESI" | Full trace |
| `ROUTER_TIMESTAMP` | datetime | 2026-01-20 18:45:32 | Timing |

2. Create **Output_Schema.xlsx**: map variables to Decipher field names + export columns

3. Verify with Bryan: which variables need to be in final data file vs QA-only?

**Deliverable:** Clean schema for QA and downstream analysis

---

## **Timeline Summary for Step 1**

| Task | Duration | Start | End | Blocker? |
|------|----------|-------|-----|----------|
| 1a: Review XMLs + quotas | 4-6 hrs | Jan 19 AM | Jan 19 PM | None |
| 1b: Map segments | 3-4 hrs | Jan 19 PM | Jan 20 AM | Needs 1a |
| 1c: Parse quota sheet | 2-3 hrs | Jan 20 AM | Jan 20 AM | Needs XLSX from Bryan |
| 1d: Map insurance logic | 2-3 hrs | Jan 20 AM | Jan 20 AM | Needs 1a |
| 1e: Termination/constraints | 2-3 hrs | Jan 20 AM | Jan 20 PM | Needs 1a |
| 1f: Pseudocode router | 3-4 hrs | Jan 20 PM | Jan 20 PM | Needs 1b, 1c, 1e |
| 1g: Logging schema | 2-3 hrs | Jan 20 PM | Jan 20 PM | Needs 1f |
| **Total** | **13-22 hrs** | **Jan 19** | **Jan 20 PM** | **Quota sheet from Bryan** |

---

## **Deliverables at End of Step 1**

1. **PRISM_System_Baseline.md** - Current state map
2. **Segment_Definition_Reference.xlsx** - All 16 segments + logic
3. **Quota_Map.md** - Current quotas + priority order
4. **Router_Logic.md** - Pseudocode + decision tree
5. **Output_Schema.xlsx** - Hidden variables + export map
6. **Test_Scenarios.md** - 5-10 test cases (all quota full, MA only, etc.)

**→ Ready to move to Step 2: Stub router XML module**