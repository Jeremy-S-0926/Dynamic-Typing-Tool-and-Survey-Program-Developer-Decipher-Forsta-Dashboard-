# Step 1e: Termination/Redirect Codes & Panel Constraints

**Status:** Complete (Jan 20, 2026)  
**Source Files:** PRISM_MA_ESI.xml, PRISM_GLP1, PRISM_AL_VAX

---

## Hard Termination Codes (Current System)

### QINSTYPE Screenout (line 921–928, PRISM_MA_ESI.xml)
- **Code:** `term_QINSTYPE`
- **Trigger:** QINSTYPE = r99 (Not answered)
- **Message:** Respondent routed to termination block
- **Exit Status:** Not answered insurance type

### Insurance Qualification Screenouts (PRISM_MA_ESI.xml)
- **Code:** `Term_QS1`, `Term_QS2` (statement qualification)
- **Trigger:** Fail qualifying statements for policy stance
- **Exit Status:** Not qualified for study content

### Party/Vote Screenouts
- **Code:** `Term_PARTYID`, `Term_VOTE24`
- **Trigger:** Party identification or 2024 vote logic
- **Status Codes:** Ideological filtering (GOP/DEM exclusive studies)

### Other Screenouts
- **Code:** `Term_QPE2`, `Term_QPE3` (professional exclusions)
- **Trigger:** Respondent in excluded professional category
- **Status Code:** Professional exclusion

---

## Current Overquota Behavior

### Status Code Assignment (lines ~2670–2700, PRISM_MA_ESI.xml)
- **Overquota MA block:** Terminates with status code (Dynata: `rst=1` for "complete", `rst=2` for "no longer eligible", `rst=3` for "over quota")
- **Overquota ESI block:** Similar status codes
- **Panel vendor:** Dynata; hard-terminates respondent (cannot re-enter same study same day)

### Exit Flow
```
respondent over-quota on selected block
  ↓
Decipher sets vscreenout marker
  ↓
Survey routes to <term> node
  ↓
Status code written to Dynata response (rst=2 or rst=3)
  ↓
Respondent removed from panel; cannot re-enter
```

---

## Router Design: Soft-Term for Overquota

### Proposed Behavior (New)
When all quota combinations are exhausted for a respondent:

1. **Typing completes normally** (XSEG_ASSIGNED set)
2. **Insurance classification completes** (XQINSTYPE set)
3. **Allocator runs** (checks all eligible blocks)
4. **All blocks full** → allocator returns ROUTER_STATUS=`OVERQUOTA_NO_ALLOCATION`
5. **Soft-term message:** "Thank you for your interest. Unfortunately, all slots for your profile are currently full. We may contact you again if new opportunities become available."
6. **Respondent exits with:** `ROUTER_STATUS=OVERQUOTA_NO_ALLOCATION`, segment tagged, but **no hard termination**
7. **Recovery:** If quotas re-open later (panel refresh), respondent remains eligible in Dynata

### Reversibility Guarantees
- **Not hard-terminated:** Respondent record remains in Dynata as "incomplete" or "no longer eligible" (soft state)
- **Recoverable:** If Block quotas are refreshed/reopened, respondent can be re-invited through a new survey link
- **Tracking:** Log entry written to QA/debug variables (ROUTER_DECISION_LOG) for audit

---

## Panel Vendor Constraints (Dynata)

### Re-entry Logic
- **Hard term (rst=3):** Respondent blocked for 24 hours or until quota reset
- **Soft term (no exit):** Respondent remains eligible but survey completion prevents re-invite in same session
- **Status field limits:** rst can carry values 0–9 (10 possible states)

### Allocation Precedence (Dynata recommendations)
- Prefer soft-term (no status code) over hard-term when possible
- Log decision path in hidden variables for later recovery
- Use `ROUTER_STATUS` tag for audit trail (not reliant on Dynata status codes)

### Time-Out Constraints
- **Session persistence:** Respondent has up to 5 minutes to complete typing + allocation
- **Survey load time:** XMLs may load slowly on mobile; allocator must complete within 30 seconds
- **Field constraints:** All hidden variables must fit within Dynata's data export (typically 255 chars per field or ~20 KB total)

---

## Hidden Variables for Termination & Recovery

### ROUTER_STATUS (primary decision flag)
| Value | Meaning |
|-------|---------|
| `SUCCESS_MA` | Allocated to MA block |
| `SUCCESS_ESI` | Allocated to ESI block |
| `SUCCESS_OTHER` | Allocated to other study (GLP1, AL, VAX) |
| `OVERQUOTA_NO_ALLOCATION` | All quotas full; soft-term |
| `TYPING_INCOMPLETE` | Typing module did not complete |
| `ERROR_INVALID_INSURANCE` | Invalid XQINSTYPE; cannot route |

### ROUTER_DECISION_LOG (audit trail)
```
Timestamp; XSEG_ASSIGNED; XQINSTYPE; INS_MA_FLAG; INS_ESI_FLAG; ESI_OPEN; MA_OPEN; DECISION; ROUTE_BLOCK; STATUS_CODE
```
Example:
```
20260119-143052; 5; 2; 0; 1; 25; 0; ESI_ONLY; ROI_ESI_FINAL; OVERQUOTA_MA_TRIGGERED
```

### ROUTER_ALLOCATION_REASON (secondary tracking)
| Reason | Meaning |
|--------|---------|
| `QUOTA_AVAILABLE` | Block had open quota; allocated |
| `QUOTA_MARGINAL` | Block at edge of quota; allocated (confirm post-completion) |
| `QUOTA_PRIORITY_TIE` | Multiple blocks equal quota; tie-break used |
| `QUOTA_FULL` | Block full; not allocated |

---

## Soft-Term Message (Respondent-Facing)

**Default message (can be customized per study):**

> "Thank you for participating in our research! Unfortunately, we've reached our target number of responses for your profile at this time. We appreciate your time and interest. If we have future opportunities that match your profile, we'll reach out to you."

**With recovery option (if approved by Bryan):**

> "Thank you for your interest! We're currently full for your profile, but we'd love to hear from you if our quotas reopen. [We may contact you again soon with another opportunity.]"

---

## Panel Vendor Coordination (Action Items for Bryan)

- [ ] **Confirm re-entry window:** Can respondents be re-invited if quotas reopen within same day/week?
- [ ] **Status code mapping:** What does Dynata expect for soft-term? (rst field, or separate completion code?)
- [ ] **Data export format:** How large can ROUTER_STATUS, ROUTER_DECISION_LOG fields be?
- [ ] **Recovery protocol:** If quotas reopen, how are previously-terminated respondents re-contacted? (new link, re-invite, auto-requalify?)
- [ ] **AL/VAX/PULSE quotas:** If these studies will route through router, provide their quota sheets + termination codes

---

## Implementation Checklist

- [ ] Implement soft-term message display (no hard exit code)
- [ ] Write ROUTER_STATUS variable to hidden field
- [ ] Write ROUTER_DECISION_LOG to debug field (truncate if needed)
- [ ] Log overquota decision to Decipher data export
- [ ] Test overquota flow with mock Dynata panel
- [ ] Verify respondent can re-enter if quotas reopen
- [ ] Coordinate with Bryan on recovery protocol + Dynata status mapping
