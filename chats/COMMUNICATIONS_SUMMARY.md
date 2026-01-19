# Client Communications Summary

Consolidated from original chat logs (Jan 12–18, 2026).

---

## Jan 12: Initial Scope & Onboarding Process

**Bryan:** Proposed project; asked for typical onboarding process.

**Jeremy:** Responded with onboarding philosophy:
- Review existing codebase to identify stable vs experimental pieces
- Identify one representative study as reference build
- Build thin end-to-end version in field early (typing + quotas + outputs)
- Iterate in short cycles with real test data
- Shift from building → repeatability once validated
- Handoff around "how to run the next study" (analyst-facing)

**Outcome:** Bryan appreciated the reliability/flexibility focus and asked for hourly rate estimate.

---

## Jan 13: Hours Estimate & Setup

**Jeremy:** Estimated 90–130 hours end-to-end.
- Lower end: clean foundation
- Upper end: hardening + surprises + dashboard integration

**Bryan:** Forwarding to finance for Upwork offer setup.

---

## Jan 17: Problem Statement & Proposed Solution

**Bryan:** Live fieldwork across 4 concurrent PRISM studies. Current problem: respondents terminated when over-quota in one study even though they could fill quotas in another = major sample waste.

**Ask:** Design + implement a minimal router so sample flows through single typing event and routes to best study based on open quotas.

**Constraint:** Need it live next week.

**Options:** Router module (pre-survey) vs single survey shell with internal versions.

**Jeremy:** Recommended Option 2 (single shell, internal versions) for time pressure + panel flow integrity.

**Proposed approach:**
- One entry point → one typing event (shared PRISM logic)
- Compute segment + priority score once
- Lightweight allocator (open quotas + priority weights + deterministic tie-break)
- Route internally to correct study block
- Soft-term if all quotas full (not hard terminate)
- Log decision path for QA

**Can implement as clean XML module, then evolve into full router later.**

**Initial focus:**
- Typing outputs
- Segment quotas per study
- Panel entry/exit constraints

**Bryan:** Shared XMLs (~5,000 lines each, bloated, no annotations). Quotas manually adjusted on fly but close to locked.

**Jeremy:** Focus on:
- Typing logic + outputs
- Quota variables / segment caps
- Entry, termination, completion logic

---

## Jan 18: Status Check

**Bryan:** Checking in on status.

**Jeremy:** Starting work immediately; will send detailed status report shortly.

---

## Summary

**Status:** Scope + approach agreed; work underway.  
**Priority:** MVP router by Jan 24 for live deployment next week.  
**Approach:** Single survey shell, soft-term on overquota, Decipher-driven quotas, QA logging.  
**Next:** Complete discovery phase, identify clarifications needed from Bryan.
