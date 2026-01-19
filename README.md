# Dynamic Typing Tool and Router (Decipher/Forsta + Dashboard)

## Project Overview

**Client:** Bryan Dumont, Reservoir Communications  
**System:** PRISM - Healthcare Policy Segmentation Research Platform  
**Stack:** Decipher/Forsta (XML + JavaScript), DisplayR Dashboards  
**Status:** Week 1 - Phase 2 Router Implementation STARTED 🚧 (Step 1 Complete ✅, ~28 hrs)  
**Timeline:** Jan 19–24 (MVP router implementation), Jan 31 (go-live)  

---

## Current Phase: Phase 2 - Router Implementation (STARTED)

### ✅ Completed Steps 1a-1h (Jan 19-20)
- **Step 1a:** Reviewed PRISM XMLs → identified `XSEG_ASSIGNED` (16 segments, stable)
- **Step 1b:** Mapped segment definitions → GOP (10) + DEM (6) scoring logic
- **Step 1c:** Parsed quota sheets → MA/ESI caps, GLP1 segment caps, all other quotas open (`inf`)
- **Step 1d:** Mapped insurance logic → QINSTYPE → XQINSTYPE → INS_MA_FLAG, INS_ESI_FLAG, INS_OTHER_FLAG
- **Step 1e:** Termination/redirect codes + panel constraints (soft-term strategy)
- **Step 1f:** Router decision tree pseudocode + flowchart (deterministic tie-break)
- **Step 1g:** Hidden variable schema (ROUTER_STATUS, ROUTER_DECISION_LOG, export format)
- **Step 1h:** Test scenarios (10 test cases covering all paths)

### ✅ Phase 2 Initial Deliverables (Jan 19-20)
- **Step 2a:** Router XML stub (`router/router_module.xml`, 370 lines)
- **Step 2b:** Design decisions doc (8 assumptions + rationale)
- **Step 2c:** Integration guide (step-by-step extraction + quota API setup)
- **Step 2d:** Test plan (10 executable test cases adapted from Step 1h)

---

## Quick Reference: Key Findings

| Item | Value |
|------|-------|
| Segments | 16 stable (GOP 1–10, DEM 11–16) |
| Typing output | `XSEG_ASSIGNED` (guaranteed completion) |
| Insurance classes | XQINSTYPE: 0=MA, 1=Traditional, 2=ESI, 3=Other |
| MA/ESI branching | XRANDOMPICK (quota-driven, marker-based) |
| Quota system | Live Decipher sheets (Block, Wave Equal, Age/Gender, Dupes, Random) |
| MA/ESI caps | Block_INSTYPE_Quota (ESI 75–750, MA 200) + Wave Equal (75/50) |
| GLP1 caps | Numeric segment caps (30–350), age/gender ranges |
| Overquota default | Soft-term + `ROUTER_STATUS=OVERQUOTA_TAGGED` |

---

## Project Structure

```
.
├── README.md                           # This file
├── PROJECT_STATUS.md                   # Detailed tracking & planning
├── chats/                              # Client communications
│   └── (consolidated from 3 chat logs)
├── source/                             # Original files (do not edit)
│   ├── PRISM_MA_ESI.xml
│   ├── PRISM_GLP1, PRISM_AL_VAX
│   ├── PRISM_Segmentation_Typing_Tools.xlsx
│   ├── MA_ESI_quota.xls, GLP1-quota.xls
├── discovery/                          # Analysis docs (Step 1)
│   ├── PRISM_System_Baseline.md        # ✅ Typing + quotas overview
│   ├── Step_1b_Segment_Map.md          # ✅ 16 segments + scoring logic
│   ├── Step_1c_Quota_Status.md         # ✅ Quota sheets unblocked
│   ├── Quota_Map.md                    # ✅ Sheet → tag mapping + caps
│   ├── Step_1d_Insurance_Logic.md      # ✅ QINSTYPE + XQINSTYPE + XRANDOMPICK
│   ├── Step_1e_Termination_Redirect.md # ✅ Codes + panel constraints
│   ├── Step_1f_Router_Logic.md         # ✅ Pseudocode + flowchart
│   ├── Step_1g_Output_Schema.md        # ✅ Hidden variables
│   └── Step_1h_Test_Scenarios.md       # ✅ Test cases (10 scenarios)
├── router/                             # Router implementation (Phase 2)
│   ├── router_module.xml               # ✅ Router XML stub (370 lines)
│   ├── DESIGN_DECISIONS.md             # ✅ Assumptions + rationale
│   ├── INTEGRATION_GUIDE.md            # ✅ Step-by-step integration
│   └── tests/
│       └── TEST_SCENARIOS.md           # ✅ 10 executable test cases
│   ├── Step_1e_Termination_Redirect.md # ✅ Codes + panel constraints
│   ├── Step_1f_Router_Logic.md         # ✅ Pseudocode + flowchart
│   ├── Step_1g_Output_Schema.md        # ✅ Hidden variables
│   └── Step_1h_Test_Scenarios.md       # ✅ Test cases (10 scenarios)
├── router/                             # Router implementation (Phase 2)
│   └── (to be created)
└── .gitignore
```

---

## Open Questions for Bryan

1. **Quota placeholders:** What do `*` (Total Quota) and `inf` mean? (no cap vs. open placeholder?)
2. **GLP1 age/gender ranges:** Are both bounds enforced (e.g., 250–450) or only upper?
3. **MA/ESI block balance:** Should Block Quota c_1/c_2 caps be set for deterministic 50/50?
4. **Study sheets:** Provide AL_VAX and other study quota sheets if in scope.
5. **Priority order:** Confirm MA > ESI > GLP1 > others or adjust.

---

## Next Actions

**Today/Tomorrow (Jan 20):**
1. Complete Steps 1e–1h
2. Compile all discovery docs + summary
3. Send clarification questions to Bryan

**Week 2 (Jan 24–31):**
1. Begin Phase 2: Stub router XML module (MVP)
2. Integrate with existing XMLs (safe skeleton)
3. Deploy to staging + QA
4. Go-live

---

## How to Navigate This Project

- **Quick status:** See `PROJECT_STATUS.md` (this file has detailed tracking)
- **Discovery findings:** See `discovery/` folder (8 docs, organized by step)
- **Questions for Bryan:** See "Open Questions" above
- **Original files:** See `source/` folder (XMLs, quota sheets, typing tools)
- **Chats/context:** See `chats/` folder (consolidated client communications)
- **Soft-term messaging:** "Thank you for your time. Our quotas have been filled. Your responses will help us build our research panel."
- **Panel vendor:** Status codes to be confirmed (overquota_tagged=true)

---

## Next Steps

1. ✅ Git repo initialized
2. ⏳ Complete Phase 1 discovery (by Jan 20)
3. ⏳ Design router pseudocode
4. ⏳ Build router XML module
5. ⏳ Test & deploy

---

**Last Updated:** Jan 19, 2026  
**Version:** 0.1 (Initial Setup)
