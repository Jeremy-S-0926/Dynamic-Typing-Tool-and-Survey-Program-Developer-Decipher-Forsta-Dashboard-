# 📋 WORKSPACE GUIDE — START HERE

**Last Updated:** Jan 20, 2026  
**Phase:** Discovery & Analysis COMPLETE ✅ (All Steps 1a–1h)  
**Status:** Ready for Phase 2 - Router XML Implementation  

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

## 🚀 What Just Completed (Jan 19)

### ✅ Step 1a: System Baseline
- Reviewed all 3 XMLs; identified typing output = `XSEG_ASSIGNED` (16 segments)
- Confirmed quotas read live from Decipher sheets
- Documented MA/ESI branching logic (XRANDOMPICK)
- Identified overquota default: soft-term + ROUTER_STATUS=OVERQUOTA_TAGGED

### ✅ Step 1b: Segment Definitions
- Listed all 16 segments (GOP 1–10, DEM 11–16)
- Documented scoring: z-scores → squared-distance to centroids → winner selection
- Verified identical logic + stability across MA/ESI, GLP1, AL_VAX

### ✅ Step 1c: Quota Sheets (UNBLOCKED!)
- Parsed MA_ESI_quota.xls: found real caps (Block_INSTYPE 75–750 ESI / 200 MA; Wave Equal 75/50)
- Parsed GLP1_quota.xls: found numeric segment caps (30–350) + age/gender ranges
- Created sheet → Decipher tag mapping
- Identified open questions (placeholder meanings, range enforcement, block balance)

### ✅ Step 1d: Insurance Logic
- Documented QINSTYPE (employer/spouse/parent/self/Medicare) + QINS_MEDICARE (Traditional/MA)
- Mapped to XQINSTYPE derivation: 0=MA, 1=Traditional, 2=ESI, 3=Other
- Documented XRANDOMPICK (MA/ESI selector when both quotas open)
- Derived eligibility flags for router: INS_MA_FLAG, INS_ESI_FLAG, INS_OTHER_FLAG

### 🧹 Workspace Cleanup
- Created `chats/` folder → consolidated 3 chat logs into COMMUNICATIONS_SUMMARY.md
- Created `source/` folder (ready to move original files)
- Created `PROJECT_STATUS.md` as single source of truth
- Updated `README.md` with quick ref table + structure
- Created `discovery/README.md` as navigation guide for all discovery docs
- Merged old "Step 1 - Discovery & Analysis Phase.md" + "To Do List.md" into PROJECT_STATUS.md

---

## 🚧 What's Next (Due Jan 20)

### Step 1e: Termination/Redirect Codes
Find all exit codes + panel constraints; design soft-term for overquota

### Step 1f: Router Decision Tree
Write full pseudocode + flowchart before coding begins

### Step 1g: Output Schema
Define all hidden variables (XSEG_ASSIGNED, STUDY_ASSIGNED, ROUTER_STATUS, decision logs, etc.)

### Step 1h: Test Scenarios
Design 5–10 representative test cases for QA validation

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
