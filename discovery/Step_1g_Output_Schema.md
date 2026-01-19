# Step 1g: Output Schema & Hidden Variables

**Status:** Complete (Jan 20, 2026)  
**Deliverable:** Complete variable schema for router implementation + data export

---

## Hidden Variable Schema (Router Implementation)

### Core Typing Output (Pre-Existing, Preserved)
| Variable | Label | Type | Rows | Purpose | Source |
|----------|-------|------|------|---------|--------|
| `XSEG_ASSIGNED` | Final Segment | Radio | 1–16 | Respondent's assigned segment (GOP 1–10 or DEM 11–16) | Typing module (exec) |
| `XGOP_SEG_FINAL_1` | GOP Winner | Radio | 1–10 | Lowest distance to GOP centroid | Typing module (exec) |
| `XGOP_SEG_FINAL_2` | GOP Runner-Up | Radio | 1–10 | Second-lowest distance to GOP | Typing module (exec) |
| `XDEM_SEG_FINAL_1` | DEM Winner | Radio | 1–6 | Lowest distance to DEM centroid | Typing module (exec) |
| `XDEM_SEG_FINAL_2` | DEM Runner-Up | Radio | 1–6 | Second-lowest distance to DEM | Typing module (exec) |

### Insurance Classification (Pre-Existing, Preserved)
| Variable | Label | Type | Values | Purpose | Source |
|----------|-------|------|--------|---------|--------|
| `XQINSTYPE` | Insurance Type (Hidden) | Radio | 0=MA, 1=Traditional, 2=ESI, 3=Other | Derived insurance classifier | Exec (QINSTYPE + QINS_MEDICARE) |
| `QINSTYPE` | Insurance Type (Response) | Radio | r1/r2/r3/r4/r7/r99 | Respondent's primary insurance source | Q885 |
| `QINS_MEDICARE` | Medicare Type | Radio | r1/r2 | Traditional or Medicare Advantage (if r7) | Q933 |

### Router Decision Output (New)
| Variable | Label | Type | Values | Purpose | Source | Export | Example |
|----------|-------|------|--------|---------|--------|--------|---------|
| `XRANDOMPICK` | MA/ESI Route | Radio | 1=ESI, 2=MA | Study branch selector (MA vs ESI) | Router allocator | **Yes** | 1 |
| `ROUTER_STATUS` | Router Decision | Radio | SUCCESS_MA, SUCCESS_ESI, OVERQUOTA_NO_ALLOCATION, TYPING_INCOMPLETE, ERROR_INVALID_INSURANCE | Final router decision (success, overquota, error) | Router (main) | **Yes** | SUCCESS_ESI |
| `ROUTER_ALLOCATION_REASON` | Allocation Reason | Radio | QUOTA_AVAILABLE, QUOTA_FULL, XQINSTYPE_OTHER_NO_ROUTE | Why was this decision made? | Router (allocator) | **Yes** | QUOTA_AVAILABLE |
| `ROUTER_DECISION_LOG` | Decision Audit Trail | Text | Semicolon-delimited log | Detailed decision path with quota states (for QA) | Router (main) | **Yes** | `20260119-143052; XSEG=5; XQINS=2; ESI_OPEN=25; MA_OPEN=0; DECISION=ROUTE; ROUTE=ROI_ESI_FINAL; STATUS=SUCCESS_ESI` |
| `ROUTE_BLOCK_INTERNAL` | Assigned Block (Internal) | Hidden | ROI_ESI_FINAL, ROI_MA_FINAL, NONE | Which survey block to route to (internal reference) | Router (allocator) | **No** | ROI_ESI_FINAL |

---

## Decipher Implementation: Hidden Question Setup

All router variables should be declared as hidden (execute only, no display):

```xml
<!-- ROUTER DECISION VARIABLES -->

<!-- Primary: Router Status Flag -->
<question label="ROUTER_STATUS" type="radio" hidden="true" export="true" execute="true">
  <option value="1" label="SUCCESS_MA">Allocated to MA</option>
  <option value="2" label="SUCCESS_ESI">Allocated to ESI</option>
  <option value="3" label="OVERQUOTA_NO_ALLOCATION">All quotas full; soft-term</option>
  <option value="4" label="TYPING_INCOMPLETE">Typing module incomplete</option>
  <option value="5" label="ERROR_INVALID_INSURANCE">Invalid insurance type</option>
</question>

<!-- Secondary: MA/ESI Selector (set by allocator) -->
<question label="XRANDOMPICK" type="radio" hidden="true" export="true" execute="true">
  <option value="1" label="ESI">Employer-sponsored</option>
  <option value="2" label="MA">Medicare Advantage or Traditional</option>
</question>

<!-- Tertiary: Reason for Decision -->
<question label="ROUTER_ALLOCATION_REASON" type="radio" hidden="true" export="true" execute="true">
  <option value="1" label="QUOTA_AVAILABLE">Block had available quota</option>
  <option value="2" label="QUOTA_FULL">Block at or over quota</option>
  <option value="3" label="XQINSTYPE_OTHER_NO_ROUTE">Insurance type not routable</option>
</question>

<!-- Audit: Decision Log (text field) -->
<question label="ROUTER_DECISION_LOG" type="text" hidden="true" export="true" execute="true" length="500">
  <!-- Contains: timestamp; XSEG; XQINS; ESI_OPEN; MA_OPEN; DECISION; ROUTE_BLOCK; STATUS -->
</question>

<!-- Internal Reference (not exported) -->
<question label="ROUTE_BLOCK_INTERNAL" type="text" hidden="true" export="false" execute="true" length="50">
  <!-- Values: ROI_ESI_FINAL, ROI_MA_FINAL, NONE -->
</question>
```

---

## Data Export Schema

### Analyst Export (Decipher Data Download)
**Fields to include in CSV/XLSX:**

| Field | Source | Format | Example | Notes |
|-------|--------|--------|---------|-------|
| `ResponseID` | Decipher native | Integer | 12345678 | Panel respondent ID |
| `XSEG_ASSIGNED` | Typing module | Integer 1–16 | 5 | Final segment |
| `QINSTYPE` | Q885 response | String (r-code) | r3 | Raw insurance type |
| `XQINSTYPE` | Exec | Integer 0–3 | 2 | Derived insurance classifier |
| `XRANDOMPICK` | Router | Integer 1–2 | 1 | MA/ESI route |
| `ROUTER_STATUS` | Router | String | SUCCESS_ESI | Decision flag |
| `ROUTER_ALLOCATION_REASON` | Router | String | QUOTA_AVAILABLE | Why allocated |
| `ROUTER_DECISION_LOG` | Router | String (delimited) | `20260119-143052; XSEG=5; ...` | Audit trail |
| `CompletionTime` | Decipher native | Datetime | 2026-01-19 14:30:52 | Survey completion timestamp |
| `Status` | Decipher native | String | Completed, Terminated | Survey outcome |

### QA/Debug Export (Optional, Decipher Debugging Only)
**Fields for post-hoc analysis:**

| Field | Format | Example | Purpose |
|-------|--------|---------|---------|
| `XGOP_RAW` | Distance array | [12.3, 45.6, 23.1, ...] | GOP segment distances (for tie verification) |
| `XDEM_RAW` | Distance array | [56.7, 89.2, ...] | DEM segment distances (for tie verification) |
| `XGOP_SEG_FINAL_1` | Integer 1–10 | 5 | GOP winner (if applicable) |
| `XGOP_SEG_FINAL_2` | Integer 1–10 | 7 | GOP runner-up (if applicable) |
| `XDEM_SEG_FINAL_1` | Integer 1–6 | 3 | DEM winner (if applicable) |
| `XDEM_SEG_FINAL_2` | Integer 1–6 | 2 | DEM runner-up (if applicable) |
| `QUOTA_STATE_AT_ALLOCATION` | JSON-like string | `{ "ESI_OPEN": 25, "MA_OPEN": 0 }` | Quota headroom at decision time |
| `ROUTER_DECISION_LOG` | String (delimited) | `20260119-143052; ...` | Full decision path |

---

## ROUTER_DECISION_LOG Format (Detailed)

**Delimiter:** Semicolon `;`  
**Order:** Timestamp; XSEG; XQINS; INS_MA; INS_ESI; ESI_OPEN; MA_OPEN; DECISION; ROUTE; STATUS  
**Max length:** 500 characters (truncate or compress if needed)

### Example Log Entries

#### Example 1: Successful MA Allocation
```
20260119-143052; XSEG=5; XQINS=0; MA=1; ESI=0; ESI_OPEN=0; MA_OPEN=15; DECISION=ROUTE; ROUTE=ROI_MA_FINAL; STATUS=SUCCESS_MA
```

#### Example 2: Successful ESI Allocation (Tie-Break)
```
20260119-143100; XSEG=6; XQINS=2; MA=0; ESI=1; ESI_OPEN=30; MA_OPEN=25; DECISION=ROUTE_TIE; ROUTE=ROI_ESI_FINAL; STATUS=SUCCESS_ESI
```

#### Example 3: Overquota Soft-Term
```
20260119-143108; XSEG=8; XQINS=1; MA=1; ESI=0; ESI_OPEN=0; MA_OPEN=0; DECISION=OVERQUOTA; ROUTE=NONE; STATUS=OVERQUOTA_NO_ALLOCATION
```

#### Example 4: Error - Typing Incomplete
```
20260119-143115; XSEG=NULL; XQINS=NULL; MA=NULL; ESI=NULL; ESI_OPEN=NULL; MA_OPEN=NULL; DECISION=ERROR; ROUTE=NONE; STATUS=TYPING_INCOMPLETE
```

---

## Integration Points with Decipher XML

### In Typing Module (post-scoring exec)
```xml
<!-- After XSEG_ASSIGNED is set -->
<exec label="router_init">
  <!-- Initialize ROUTER_STATUS to PENDING -->
  ROUTER_STATUS.setValue('pending')
</exec>
```

### In Insurance Classification (post-XQINSTYPE derivation)
```xml
<!-- After XQINSTYPE is set -->
<exec label="validate_insurance">
  <!-- Validate XQINSTYPE is 0–3; if invalid, mark ERROR -->
  if (XQINSTYPE == null || XQINSTYPE > 3) {
    ROUTER_STATUS.setValue('ERROR_INVALID_INSURANCE')
  }
</exec>
```

### In Router Allocator (main decision block)
```xml
<!-- Allocator exec (calls router logic) -->
<exec label="route_allocate">
  <!-- Pseudocode: read quotas, call router_pseudocode() -->
  ESI_OPEN = Block_INSTYPE_Quota[XQINSTYPE]['ESI_limit'] - Block_INSTYPE_Quota[XQINSTYPE]['ESI_current']
  MA_OPEN = Block_INSTYPE_Quota[XQINSTYPE]['MA_limit'] - Block_INSTYPE_Quota[XQINSTYPE]['MA_current']
  
  if (ESI_OPEN > 0 && MA_OPEN > 0) {
    XRANDOMPICK = (XSEG_ASSIGNED % 2) + 1  // tie-break: odd→1, even→2
  } else if (ESI_OPEN > 0) {
    XRANDOMPICK = 1
  } else if (MA_OPEN > 0) {
    XRANDOMPICK = 2
  } else {
    ROUTER_STATUS.setValue('OVERQUOTA_NO_ALLOCATION')
    return
  }
  
  // Set ROUTE and STATUS
  if (XRANDOMPICK == 1) {
    ROUTE_BLOCK_INTERNAL = 'ROI_ESI_FINAL'
    ROUTER_STATUS.setValue('SUCCESS_ESI')
  } else {
    ROUTE_BLOCK_INTERNAL = 'ROI_MA_FINAL'
    ROUTER_STATUS.setValue('SUCCESS_MA')
  }
  
  // Write audit log
  timestamp = getCurrentTimestamp()
  ROUTER_DECISION_LOG.setValue(
    timestamp + '; XSEG=' + XSEG_ASSIGNED + '; XQINS=' + XQINSTYPE + 
    '; ESI_OPEN=' + ESI_OPEN + '; MA_OPEN=' + MA_OPEN + 
    '; DECISION=ROUTE; ROUTE=' + ROUTE_BLOCK_INTERNAL + '; STATUS=' + ROUTER_STATUS
  )
</exec>
```

### Conditional Routing (post-allocator)
```xml
<!-- Route based on XRANDOMPICK -->
<block label="b1" cond="XRANDOMPICK == 1">
  <!-- ESI content -->
</block>

<block label="b3" cond="XRANDOMPICK == 2">
  <!-- MA content -->
</block>

<block label="b_softterm" cond="ROUTER_STATUS == 'OVERQUOTA_NO_ALLOCATION'">
  <!-- Soft-term message (no hard exit) -->
</block>
```

---

## Data Dictionary (For Analysts)

| Variable | Definition | Valid Range | Null Meaning | Example |
|----------|------------|-------------|--------------|---------|
| XSEG_ASSIGNED | Final segment assignment based on typing scoring distance to centroids | 1–16 | Typing incomplete | 5 |
| XQINSTYPE | Insurance classification derived from QINSTYPE + QINS_MEDICARE | 0–3 | Insurance question not answered | 2 |
| XRANDOMPICK | MA/ESI route selected by allocator based on quota availability | 1–2 | No quota available or error | 1 |
| ROUTER_STATUS | Final router decision flag | {SUCCESS_MA, SUCCESS_ESI, OVERQUOTA_NO_ALLOCATION, TYPING_INCOMPLETE, ERROR_INVALID_INSURANCE} | Respondent incomplete | SUCCESS_ESI |
| ROUTER_ALLOCATION_REASON | Why the router made this decision | {QUOTA_AVAILABLE, QUOTA_FULL, XQINSTYPE_OTHER_NO_ROUTE} | Router error | QUOTA_AVAILABLE |
| ROUTER_DECISION_LOG | Detailed audit trail of router decision with quota states at time of allocation | Semicolon-delimited string (max 500 chars) | Allocator not run | `20260119-143052; XSEG=5; ...` |

---

## Validation Checklist

- [ ] All hidden variables declared in XML with `hidden="true"` and `export="true"`
- [ ] ROUTER_STATUS radio options match pseudocode values
- [ ] ROUTER_DECISION_LOG field max length sufficient (500 chars = ~80 character entries)
- [ ] Exec assignments verify XSEG_ASSIGNED and XQINSTYPE before proceeding
- [ ] Soft-term block triggered by ROUTER_STATUS == OVERQUOTA_NO_ALLOCATION
- [ ] Data export includes XSEG_ASSIGNED, XQINSTYPE, XRANDOMPICK, ROUTER_STATUS, ROUTER_DECISION_LOG
- [ ] QA fields (XGOP_RAW, XDEM_RAW, QUOTA_STATE) available for debugging but not in standard export
- [ ] No hard termination codes written to Dynata status field (soft-term only)
