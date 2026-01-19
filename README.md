# Dynamic Typing Tool and Survey Program Developer (Decipher/Forsta + Dashboard)

## Project Overview

**Client:** Bryan Dumont, Reservoir Communications  
**System:** PRISM - Healthcare Policy Segmentation Research Platform  
**Stack:** Decipher/Forsta (XML + JavaScript), DisplayR Dashboards  
**Status:** In Development - Week 1 (Router MVP)

---

## Immediate Deliverable (Week 1)

### Minimal Viable Router
- Single entry point with universal typing
- Study assignment based on segment + open quotas
- Soft-term with segment tagging for overquota respondents
- QA logging for all routing decisions

**Timeline:** Jan 19-24, 2026

---

## Project Structure

```
.
├── README.md                          # This file
├── PRISM_System_Baseline.md          # Current system analysis
├── client_files/                      # Original XMLs from gists
│   ├── PRISM_MA_ESI.xml
│   ├── PRISM_GLP1
│   └── PRISM_AL_VAX
├── research/                          # Discovery & analysis docs
│   ├── Segment_Definition_Reference.xlsx
│   ├── Quota_Map.md
│   ├── Router_Logic.md
│   └── Output_Schema.xlsx
├── router/                            # Router implementation (Step 2+)
│   ├── prism_router_module.xml
│   ├── router_tests.md
│   └── integration_points.md
├── docs/                              # Project documentation
│   ├── Step_1_Discovery.md
│   ├── Architecture.md
│   └── Setup_Guide.md
└── .gitignore
```

---

## Phases

### Phase 1: Discovery & Analysis (Week 1 - 13-22 hrs)
- [ ] Review existing PRISM XML + quotas
- [ ] Map segment definitions and typing algorithm
- [ ] Parse current quota sheet
- [ ] Map insurance classification logic
- [ ] Identify termination/redirect codes
- [ ] Design router decision tree (pseudocode)
- [ ] Build logging/hidden variable schema

**Status:** In Progress

### Phase 2: Router MVP Implementation (Week 1-2 - 15-20 hrs)
- [ ] Stub router XML module (safe skeleton)
- [ ] Integration points mapping + test plan
- [ ] Deploy router to staging + QA
- [ ] Expand router to remaining studies
- [ ] Field validation + go-live

**Status:** Pending Phase 1 completion

### Phase 3: Core Architecture Stabilization (Weeks 3-4 - 30-40 hrs)
- [ ] Modularize core logic
- [ ] Reusable XML templates
- [ ] Shared JavaScript utilities
- [ ] Documentation

### Phase 4: MaxDiff Templating (Weeks 5-6 - 25-35 hrs)
- [ ] External design file ingestion
- [ ] Automated task construction
- [ ] Scoring output generation

### Phase 5: Configuration Layer (Weeks 6-7 - TBD)
- [ ] Analyst-facing template approach
- [ ] Variable/text/list substitution
- [ ] Multi-wave support

### Phase 6: ROI Dashboard (Weeks 7-8 - 15-25 hrs)
- [ ] DisplayR or alternative setup
- [ ] Real-time data pipeline
- [ ] Client-facing visualizations

---

## Key Variables & Outputs

### Router Output Schema
| Variable | Type | Purpose |
|----------|------|---------|
| `XSEG_ASSIGNED` | string | Primary segment (16 possible values) |
| `STUDY_ASSIGNED` | string | MA \| ESI \| GLP1 \| PULSE_AL \| PULSE_VAX \| OVERQUOTA_TAGGED |
| `STUDY_INTENT` | string | Which study would have taken them |
| `ROUTER_STATUS` | string | ASSIGNED \| OVERQUOTA_TAGGED |
| `INS_MA_FLAG` | binary | Medicare Advantage eligible |
| `INS_ESI_FLAG` | binary | Employer-sponsored insurance eligible |
| `ROUTER_DECISION_PATH` | text | Full trace for QA |

---

## Client Contact

**Name:** Bryan Dumont  
**Company:** Reservoir Communications  
**Current Studies:**
- Medicare Advantage (MA)
- Employer-Sponsored Insurance (ESI)
- GLP-1 Compounding
- PULSE_AL (Pharma Industry Investment)
- PULSE_VAX (Vaccination Programs)

---

## Technical Notes

- **Segments:** ~16 stable segments used across all studies
- **Quotas:** Managed via Decipher quota sheets (live reads)
- **Priority:** MA > ESI > GLP1 > others (TBD)
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
