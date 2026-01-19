Based on the client conversations and requirements, here's the complete project breakdown:

## Project Overview

**Client:** Bryan Dumont, [[Reservoir]] Communications  
**System:** [[PRISM - Healthcare policy segmentation research platform  ]]
**Stack:** Decipher/Forsta (XML + JavaScript), DisplayR dashboards  
**[[Context]]:** Live multi-study [[fielding system]] with ~16 stable segments across concurrent healthcare policy studies

---

## Immediate [[Crisis]] (Week 1 - URGENT)

**Problem:** Four studies running concurrently next week. Respondents terminated when over-quota in one study even though they could fill quotas in another = massive sample waste.

**Deliverable:** Minimal viable router (live by next week)

### Router Requirements:
1. **[[Single entry point]]** → one typing event (shared PRISM logic)
2. **Universal screener** collects:
   - Pregnancy/[[family status]] (QPREG → PREGNANCY_FLAG)
   - Voter registration (QS1)
   - [[2024 vote (QVOTE24)]]
   - [[Party ID (QS5-QS5A-QS5B)]]
   - Age (QAGE)
   - Gender (QGENDER)
   - ZIP (QZIP)

1. **Typing tool** [[assigns]]:
   - Primary segment → `XSEG_ASSIGNED` (stable across studies)
   - Future: [[segment probabilities/top-N]]

4. **Insurance classification** sets flags:
   - `INS_ESI_FLAG` (employer-sponsored insurance)
   - `INS_MA_FLAG` (Medicare Advantage)
   - `INS_OTHER_FLAG`

5. **Route eligibility** (non-exclusive booleans):
   - `ELIG_PREGNANCY`
   - `ELIG_MA`
   - `ELIG_ESI`
   - `ELIG_GLP1`
   - `ELIG_PULSE_AL`
   - `ELIG_PULSE_VAX`

6. **Router decision** (single assignment):
   - Input: `XSEG_ASSIGNED` + insurance flags + open quotas + study priority
   - Output: `STUDY_ASSIGNED` ∈ {PREG, MA, ESI, GLP1, PULSE_AL, PULSE_VAX}
   - Logic priority:
     - Primary: segment quota headroom per study
     - Tiebreaker: study priority weight
     - Secondary: deterministic (segment ID mod or respondent ID mod)

7. **MA/ESI special case:** Mutually exclusive per respondent. Priority weight breaks ties when both open.

8. **Overquota handling:**
   - Don't terminate - tag with `XSEG_ASSIGNED` + `ROUTER_STATUS=OVERQUOTA_TAGGED`
   - Persist `STUDY_INTENT` (which study they would have gone to)
   - Soft-term with polite message
   - Exit with segment intact for panel build + downstream voter/media file scoring

9. **Quota management:**
   - Read live from Decipher quota sheets
   - Future: real-time updates via lightweight admin block
   - V1: manual sheet edits acceptable

10. **Logging (QA/debugging):**
    - `XSEG_ASSIGNED`
    - `STUDY_ASSIGNED` or `STUDY_INTENT`
    - `ROUTER_STATUS` (ASSIGNED | OVERQUOTA_TAGGED)
    - Eligibility flags evaluated
    - Quotas checked
    - Decision path

---

## Long-term System (90-130 hours total)

### 1. **Core Architecture Stabilization**
- **Current state:** Three 5,000+ line XML files (bloated, no annotations)
  - `PRISM_MA_ESI.xml`
  - `PRISM_GLP1`
  - `PRISM_AL_VAX`
- **Goal:** Modular, reusable, maintainable architecture
- **Deliverables:**
  - Separate core logic from configuration
  - Consistent naming conventions
  - Reusable XML modules
  - Shared JavaScript utility files
  - Documentation

### 2. **Dynamic Typing Tool Algorithm**
- **Inputs:** Weighted survey items, indices, composite scores
- **Outputs:**
  - Primary segment assignment (`XSEG_ASSIGNED`)
  - Confidence/probability scores
  - Distance-to-segment metrics
  - Top-N segment assignments (future)
- **Logic:**
  - Rules-based + probabilistic scoring
  - Tie-breaking (deterministic)
  - Ambiguity handling
  - Transparent decision logging to hidden variables
- **QA:** Full decision trace for post-field audits

### 3. **Dynamic Sample Management**
- **Quotas:**
  - By segment (16 segments)
  - By key demographics
  - Real-time quota state monitoring
- **Routing:**
  - Smart assignment based on open quotas
  - Soft/hard termination logic
  - Panel vendor compatibility (entry links, redirects, status codes)
- **Priority system:**
  - Study-level weights
  - Segment-level ROI scores
  - Dynamic reallocation as quotas fill

### 4. **MaxDiff Templating**
- **Current pain point:** Custom rebuilds every wave
- **Goal:** Automated, repeatable system
- **Features:**
  - External design file ingestion
  - JavaScript parsing
  - Dynamic task construction
  - Controlled randomization
  - Version control
  - Scoring outputs as survey variables
  - Analysis-ready exports (no manual post-processing)

### 5. **Configuration Layer (Analyst-Facing)**
- **Goal:** Analysts launch studies without touching core code
- **Approach:**
  - Structured configuration file OR admin UI
  - Easy variable/text/list substitution
  - Multiple versions/waves support
  - MaxDiff design file upload
  - Study priority weights
  - Quota adjustments
- **Documentation:** "How to run the next study" guide

### 6. **ROI Dashboard**
- **Platform:** DisplayR (or propose alternative with justification)
- **Data flow:** Survey outputs → API → dashboard
- **Content:**
  - Segmentation results
  - MaxDiff utilities
  - ROI metrics
  - Confidence/ambiguity scores (typing diagnostics)
  - Client-facing narratives
- **Design:** Based on existing wireframes (to be provided)
- **Requirement:** Real-time or near-real-time refresh

### 7. **Handoff Package**
- Modular codebase with documentation
- Configuration examples
- QA logging guide
- MaxDiff setup guide + sample files
- Dashboard data requirements doc
- "How to run the next study" analyst guide
- Code comments where appropriate (not exhaustive)

---

## Technical Constraints

1. **Live field risk:** Changes must be safe, reversible, minimal surface area
2. **Testing mode detection:** Code respects `gv.survey.root.state.testing`
3. **Device detection:** Desktop vs mobile logic already in place
4. **Existing utilities:** Prompt functions, Punch functions, zipCode file handling
5. **Quota sheets:** Decipher native (`<quota label="quo1" overquota="noqual" sheet="Total Quota"/>`)
6. **Panel flow:** Must preserve standard entry/exit/redirect patterns

---

## Upcoming Studies (Context for Router)

**Live now:**
- Medicare Advantage (MA)
- Employer-sponsored insurance (ESI)
- GLP-1 compounding

**Next week:**
- Pharma industry investment in American manufacturing
- Childhood vaccine schedule
- Vaccine injury compensation program (PULSE_AL)
- Vaccination for new/pregnant parents (PULSE_VAX)

---

## Success Criteria

1. **Router (Week 1):**
   - Zero sample waste from overquota terms
   - All respondents typed and tagged
   - Safe, logged, reversible

2. **Full system:**
   - Analysts can launch new studies independently
   - MaxDiff setup < 30 minutes per study
   - Typing logic transparent and auditable
   - Quotas adjustable in real-time
   - Dashboard auto-refreshes with clean metrics
   - System stable under concurrent 4+ study load

---

## Client Priorities (in order)

1. **Reliability** > analyst UX
2. **Controlled flexibility** for inevitable per-study customization
3. **Speed to field** (router needed immediately)
4. **Repeatability** (templated, not bespoke)
5. **Transparency** (QA logging, decision traces)
6. **ROI clarity** for clients

---

## Timeline Estimate

- **Week 1:** Minimal router (20-30 hours)
- **Weeks 2-4:** Core stabilization + typing tool hardening (30-40 hours)
- **Weeks 5-6:** MaxDiff templating + configuration layer (25-35 hours)
- **Weeks 7-8:** Dashboard + handoff (15-25 hours)

**Total:** 90-130 hours (lower = clean foundation; upper = surprises + refactor)