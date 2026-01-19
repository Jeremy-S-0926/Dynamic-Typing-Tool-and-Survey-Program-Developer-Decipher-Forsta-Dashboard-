# 📋 WORKSPACE GUIDE — START HERE

**Last Updated:** Jan 20, 2026  
**Phase:** Phase 2 - Router Implementation ✅ Testing Complete (Step 1 Complete)  
**Status:** Router fixed and retested (10/10 pass); pending integration + staging + Bryan review  

---

## 🎯 Quick Navigation

### 📊 Current Status & Planning
- **PRIMARY:** [`PROJECT_STATUS.md`](PROJECT_STATUS.md) — Master tracking document (timeline, completed tasks, pending work, open questions)
- **DETAILED:** [`WORK_REPORT_Jan19.md`](WORK_REPORT_Jan19.md) — Full day's findings + discoveries (detailed summary for review)

### 📁 Discovery Documents (Analysis Results)
All findings organized in [`discovery/`](discovery/) folder:

| Document | Status | Content |
|----------|--------|---------|
| [`PRISM_System_Baseline.md`](discovery/PRISM_System_Baseline.md) | ✅ | Current typing + quotas + routing overview |
| [`Step_1b_Segment_Map.md`](discovery/Step_1b_Segment_Map.md) | ✅ | 16 segments + scoring logic (stable across all studies) |
| [`Step_1c_Quota_Status.md`](discovery/Step_1c_Quota_Status.md) | ✅ | Quota sheets unblocked + open questions |
| [`Quota_Map.md`](discovery/Quota_Map.md) | ✅ | Sheet → Decipher tag mapping + numeric caps |
| [`Step_1d_Insurance_Logic.md`](discovery/Step_1d_Insurance_Logic.md) | ✅ | Insurance classification + XQINSTYPE + XRANDOMPICK |
| [`Step_1e_Termination_Redirect.md`](discovery/Step_1e_Termination_Redirect.md) | ✅ | Exit codes + panel constraints + soft-term strategy |
| [`Step_1f_Router_Logic.md`](discovery/Step_1f_Router_Logic.md) | ✅ | Complete pseudocode + flowchart + tie-break logic |
| [`Step_1g_Output_Schema.md`](discovery/Step_1g_Output_Schema.md) | ✅ | Hidden variables + data export schema + Decipher XML setup |
| [`Step_1h_Test_Scenarios.md`](discovery/Step_1h_Test_Scenarios.md) | ✅ | 10 test cases covering all router paths |

### 🔧 Router Implementation (Phase 2)
| Document | Status | Content |
|----------|--------|---------|  
| [`router/router_module.xml`](router/router_module.xml) | ✅ | Router XML (fixed, 407 lines, all sections) |
| [`router/DESIGN_DECISIONS.md`](router/DESIGN_DECISIONS.md) | ✅ | 8 assumptions + rationale + risk assessment |
| [`router/INTEGRATION_GUIDE.md`](router/INTEGRATION_GUIDE.md) | ✅ | Step-by-step integration + quota API setup |
| [`router/tests/TEST_SCENARIOS.md`](router/tests/TEST_SCENARIOS.md) | ✅ | 10 executable test cases + validation checklist |
| [`router/VALIDATION_REPORT.md`](router/VALIDATION_REPORT.md) | ✅ | Full QA report (all issues resolved) |
| [`router/TESTING_SUMMARY.md`](router/TESTING_SUMMARY.md) | ✅ | Client-ready testing summary |

### 💬 Client Communications
- [`chats/COMMUNICATIONS_SUMMARY.md`](chats/COMMUNICATIONS_SUMMARY.md) — Consolidated from 3 chat logs (Jan 12–18)

### 📚 Source Files (Original, Do Not Edit)
- `PRISM_MA_ESI.xml` — Medicare Advantage study XML (2,500 lines)
- `PRISM_GLP1` — GLP-1 variant XML (2,000 lines)
- `PRISM_AL_VAX` — Vaccine variant XML (2,200 lines)
- `MA_ESI_quota.xls` — MA/ESI quota sheets (25 sheets)
- `GLP1-quota.xls` — GLP1 quota sheets (19 sheets)
- `PRISM Segmentation Typing Tools.xlsx` — Typing design matrices

---

### 🚀 Recent Activity

- Fixed critical router issues (XQINSTYPE values, XRANDOMPICK, datetime import, log size, dispatch)
- Added QA docs: `router/VALIDATION_REPORT.md`, `router/TESTING_SUMMARY.md`
- Retested all 10 scenarios → **10/10 PASS**

### ✅ Step 1 Complete (Jan 19-20)
- All 8 discovery tasks finished; pseudocode, hidden schema, and test scenarios defined

### ✅ Phase 2 Progress (Jan 19-20)
- Router XML built and validated
- Design decisions + integration guide complete
- Test plan ready; validation completed

### 🔜 Up Next (Step 2e-2h)
1) Integrate router into source XMLs (extract typing, insurance, study blocks; replace quota placeholders)  
2) Deploy to staging and run 10 test cases; verify quota decrements + decision logs  
3) Send to Bryan for review/approvals; apply feedback  
4) Production deployment and monitoring (target Jan 24)  

---

## 📌 Key Findings (Tldr)

| Finding | Value |
|---------|-------|
| **Segment Output** | `XSEG_ASSIGNED` (16 stable segments) |
| **Typing Guarantee** | No termination between scoring + assignment → always completes |
| **MA/ESI Branching** | XQINSTYPE + XRANDOMPICK (quota-driven) |
| **MA/ESI Caps** | Block_INSTYPE_Quota (ESI 75–750, MA 200) + Wave Equal (75/50) |
| **GLP1 Caps** | Numeric per segment (30–350), age/gender ranges |
| **Insurance Flags** | INS_MA (XQINSTYPE ∈ {0,1}), INS_ESI (==2), INS_OTHER (==3) |
| **Overquota Default** | Soft-term + ROUTER_STATUS=OVERQUOTA_TAGGED (recoverable) |
| **Quota System** | Live Decipher reads (no local counts) |

---

## ❓ Open Questions for Bryan

1. Quota placeholders: What do `*` and `inf` mean?
2. GLP1 age/gender: are min-max ranges both enforced?
3. Block balance: should MA/ESI 50/50 be deterministic or dynamic?
4. AL_VAX sheets: provide if in scope
5. Study priority: confirm MA > ESI > GLP1 > others

See `PROJECT_STATUS.md` for full list.

---

## 📂 File Organization

```
root/
├── README.md                           # Main overview
├── PROJECT_STATUS.md ⭐                # MASTER TRACKING (timeline, tasks, questions)
├── WORK_REPORT_Jan19.md                # Detailed findings (daily summary)
├── discovery/                          # All analysis docs (8 docs, well-organized)
│   ├── README.md                       # Navigation guide for discovery docs
│   ├── PRISM_System_Baseline.md        # ✅
│   ├── Step_1b_Segment_Map.md          # ✅
│   ├── Step_1c_Quota_Status.md         # ✅
│   ├── Quota_Map.md                    # ✅
│   ├── Step_1d_Insurance_Logic.md      # ✅
│   ├── Step_1e_Termination_Redirect.md # 🚧 Due Jan 20
│   ├── Step_1f_Router_Logic.md         # 🚧 Due Jan 20
│   ├── Output_Schema.md                # 🚧 Due Jan 20
│   └── Test_Scenarios.md               # 🚧 Due Jan 20
├── chats/                              # Client communications
│   └── COMMUNICATIONS_SUMMARY.md       # Consolidated (Jan 12–18)
├── source/                             # Original files (do not edit)
│   ├── PRISM_MA_ESI.xml
│   ├── PRISM_GLP1, PRISM_AL_VAX
│   ├── MA_ESI_quota.xls, GLP1-quota.xls
│   └── PRISM_Segmentation_Typing_Tools.xlsx
└── router/                             # (Phase 2, not started)
```

---

## ⏱️ Timeline

| Phase | Tasks | Status | When |
|-------|-------|--------|------|
| **1a–1d** | System baseline, segments, quotas, insurance | ✅ | Jan 19 (~16 hrs) |
| **1e–1h** | Termination, router logic, output schema, tests | 🚧 | Jan 20 (expect 8–12 hrs) |
| **Phase 2** | Stub router XML module (MVP) | ⏳ | Jan 24–31 |
| **Go-live** | Deploy to production | ⏳ | ~Jan 31 |

---

## 💡 How to Use This Workspace

1. **First time?** Start with [`PROJECT_STATUS.md`](PROJECT_STATUS.md) for overview + timeline
2. **Daily standup?** Check `PROJECT_STATUS.md` progress table
3. **Deep dive on system?** Read [`PRISM_System_Baseline.md`](discovery/PRISM_System_Baseline.md)
4. **Understand segments?** Read [`Step_1b_Segment_Map.md`](discovery/Step_1b_Segment_Map.md)
5. **Understand routing?** Read [`Step_1d_Insurance_Logic.md`](discovery/Step_1d_Insurance_Logic.md) + (upcoming) [`Step_1f_Router_Logic.md`](discovery/Step_1f_Router_Logic.md)
6. **Full detailed summary?** Read [`WORK_REPORT_Jan19.md`](WORK_REPORT_Jan19.md)
7. **Find something?** Use [`discovery/README.md`](discovery/README.md) as index

---

## 📞 Questions?

See `PROJECT_STATUS.md` → **"Open Questions for Bryan"** section.

**Contact:** Bryan Dumont (Reservoir Communications) via Upwork.

---

**Last Check:** Jan 19, 2026 | **Next Update:** Jan 20 (Steps 1e–1h)
