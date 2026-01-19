# Router Design Decisions & Assumptions

**Status:** MVP Implementation (Phase 2a)  
**Date:** Jan 19, 2026  
**Target:** Go-live Jan 24, 2026

---

## Decision Summary

All decisions below are based on existing XML behavior, standard Decipher practices, and Bryan's requirement: **"route sample to best study based on open quotas, soft-term if all full"**. These assumptions are flagged in code comments for easy adjustment after client review.

---

## 1. Quota Placeholder Interpretation

### Decision
- **`*` (Total Quota) = unlimited/no cap**
- **`inf` (all other quotas) = unlimited/no cap**

### Rationale
- Standard Decipher convention for "open" quotas
- Bryan said "quotas manually adjusted on fly but close to locked" → these are placeholders
- Current XMLs treat these as uncapped (no overquota behavior)

### Implementation Impact
- Router reads live quota state from Decipher sheets via `<quota>` tag
- Does NOT hard-code any numeric caps
- `*` and `inf` treated identically (no enforced limit)

### Risk: **ZERO** (maintains existing behavior)

---

## 2. GLP1 Age/Gender Ranges

### Decision
**Treat as soft targets; enforce upper bounds only (not minimums)**

**Example:**
- Age 18–29: `250-450` → soft minimum 250, hard maximum 450
- Gender Male: `1100-1500` → soft minimum 1100, hard maximum 1500

### Rationale
- Enforcing hard minimums would cause survey delays (can't complete until min reached)
- Ranges like "250-450" are typical soft targets for balanced samples
- Decipher `overquota="noqual"` behavior suggests upper bounds are enforced

### Implementation Impact
- Router checks if respondent fits age/gender category
- Routes if quota NOT at upper bound
- Does NOT block routing if minimum not yet reached

### Risk: **LOW** (standard practice; can add minimum enforcement later if Bryan requests)

---

## 3. MA/ESI Block Balance (c1/c2)

### Decision
**Keep existing marker-based logic (current system behavior)**

**Current behavior:**
```
Block Quota: c_1=inf, c_2=inf (no numeric caps)
When both MA & ESI quotas open:
  - Use Block Quota cell markers to pick 50/50 (or weighted)
  - Marker assignment: hasMarker('c_1') or hasMarker('c_2')
```

### Rationale
- Current XMLs already use this logic (~line 2608-2633 PRISM_MA_ESI.xml)
- Changing it risks breaking existing panel flow
- Provides flexibility for Bryan to adjust balance via Decipher sheet markers

### Implementation Impact
- Router preserves existing `XRANDOMPICK` logic
- Deterministic tie-break option available as fallback: `XSEG_ASSIGNED % 2`
- Inline comment flags this for Bryan's review

### Risk: **ZERO** (maintains continuity)

---

## 4. Study Scope & Priority Order

### Decision
**Architecture supports 4 studies (MA, ESI, GLP1, AL_VAX); MVP implements MA/ESI + GLP1**

**Priority logic:**
```
if XQINSTYPE in {0, 1, 2}:  # MA, Traditional, or ESI
    Priority 1: Check MA/ESI quotas (insurance-restricted studies)
    if MA or ESI has open quota:
        route to MA/ESI
    else:
        Priority 2: Check GLP1 quotas (no insurance restriction)
        if GLP1 has open quota:
            route to GLP1
        else:
            SOFT_TERM (all quotas full)
else:  # XQINSTYPE = 3 (Other/no insurance)
    Priority 1: Check GLP1 only (not eligible for MA/ESI)
    if GLP1 has open quota:
        route to GLP1
    else:
        SOFT_TERM
```

### Rationale
- **Bryan's core problem:** "respondents terminated when over-quota in one study even though they could fill quotas in another"
- **Solution:** Route to constrained studies first (MA/ESI have insurance restrictions), then overflow to unconstrained (GLP1)
- **Efficiency:** Maximizes sample utilization
- **AL_VAX:** Architecture supports adding it once Bryan provides quota sheet

### Implementation Impact
- Router checks eligibility + quotas in priority order
- GLP1 acts as "catch-all" for overflow respondents
- Modular design: adding AL_VAX requires only new quota sheet + 1 priority tier

### Risk: **MEDIUM** (but this is exactly what Bryan described; easy to adjust priority order if needed)

---

## 5. AL_VAX/PULSE Integration

### Decision
**Build architecture to support AL_VAX; do NOT implement until Bryan provides quota sheet**

### Rationale
- Bryan mentioned "4 concurrent studies" in chat
- We have AL_VAX XML but no quota sheet (GLP1-quota.xls exists, but no AL_VAX-quota.xls)
- Don't guess quota structure; wait for Bryan's data

### Implementation Impact
- Router code structure allows adding AL_VAX as 4th study
- Placeholder comments in code: `// AL_VAX: Add when quota sheet provided`
- Integration takes ~1 hour once quota sheet available

### Risk: **ZERO** (conservative approach; no wasted effort)

---

## 6. Soft-Term Strategy

### Decision
**No hard termination on overquota; preserve respondent eligibility in panel**

**Behavior:**
```
All quotas full for respondent profile
  ↓
ROUTER_STATUS = OVERQUOTA_NO_ALLOCATION
ROUTER_DECISION_LOG = (detailed audit trail)
  ↓
Polite message: "Thank you. All slots currently full. We may contact you again."
  ↓
Exit with status (NOT Dynata rst=3 "over quota")
  ↓
Respondent remains eligible for future invites
```

### Rationale
- Bryan's requirement: "soft-term if all quotas full (not hard terminate)"
- Preserves sample for quota refresh/panel re-entry
- Avoids Dynata 24-hour re-entry block

### Implementation Impact
- No `<term>` nodes for overquota
- Exit via suspend/complete with ROUTER_STATUS flag
- QA can filter overquota records via ROUTER_DECISION_LOG

### Risk: **ZERO** (explicitly requested by Bryan)

---

## 7. Decision Logging & QA

### Decision
**Comprehensive audit trail for all router decisions**

**Variables (per Step 1g):**
- `ROUTER_STATUS`: Final decision (SUCCESS_MA, SUCCESS_ESI, SUCCESS_GLP1, OVERQUOTA_NO_ALLOCATION, TYPING_INCOMPLETE, ERROR_INVALID_INSURANCE)
- `ROUTER_ALLOCATION_REASON`: Why? (QUOTA_AVAILABLE, QUOTA_FULL, XQINSTYPE_OTHER_NO_ROUTE)
- `ROUTER_DECISION_LOG`: Semicolon-delimited log with timestamp, segment, insurance, quota states, decision path

**Example log:**
```
20260119-143052; XSEG=5; XQINS=2; ESI_OPEN=25; MA_OPEN=0; GLP1_OPEN=100; DECISION=ROUTE; ROUTE=ROI_ESI_FINAL; STATUS=SUCCESS_ESI
```

### Rationale
- Bryan's XMLs are "bloated, no annotations" → need visibility into router behavior
- QA can trace any routing decision
- Debugging/validation easier with complete audit trail

### Implementation Impact
- All hidden variables exported to data file
- Log written at end of router execution
- No PII in log (only codes + quota states)

### Risk: **ZERO** (QA best practice)

---

## 8. Tie-Breaking Logic

### Decision
**Deterministic mod-based tie-break when both MA & ESI quotas open**

**Logic:**
```
if ESI_OPEN > 0 AND MA_OPEN > 0:
    if XSEG_ASSIGNED % 2 == 1:  # odd segments
        XRANDOMPICK = 1 (ESI)
    else:  # even segments
        XRANDOMPICK = 2 (MA)
```

**Alternative (current system):**
- Use Block Quota markers (c_1/c_2) for 50/50 split
- Comment in code: `// Option: Use hasMarker('c_1') for random 50/50`

### Rationale
- Deterministic = reproducible for testing
- Balances MA/ESI allocation by segment
- Can switch to marker-based if Bryan prefers randomization

### Implementation Impact
- Inline code comment flags both options
- Bryan can choose during review

### Risk: **LOW** (both methods achieve 50/50 balance over time)

---

## Review Process

**For Bryan's review:**
1. All assumptions flagged in code with `// ASSUMPTION:` comments
2. This document provided alongside router stub
3. Each decision includes:
   - What we decided
   - Why (rationale)
   - Risk level (ZERO/LOW/MEDIUM)
   - How to change it if needed

**Adjustment timeline:**
- Simple changes (tie-break method, priority order): ~15 min
- Moderate changes (add AL_VAX study): ~1 hour
- Complex changes (hard quota minimums): ~2-3 hours

---

## Next Steps

1. Bryan reviews this document + router stub
2. Provides feedback on any decisions to change
3. Supplies AL_VAX quota sheet if in scope
4. Approves for staging deployment

**Target:** Feedback by Jan 21, deploy to staging Jan 22-23, go-live Jan 24
