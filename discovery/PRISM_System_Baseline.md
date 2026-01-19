# PRISM System Baseline

## Variable Map (key exported/hidden vars)
- `XSEG_ASSIGNED` (radio, hidden, execute/survey/report): final 1–16 segment. Set from `XGOP_SEG_FINAL_1` or `XDEM_SEG_FINAL_1+10` (~line 2551 PRISM_MA_ESI.xml).
- `XQINSTYPE` (radio, hidden): insurance class derived from `QINSTYPE` + `QINS_MEDICARE` (~line 2584). Values: r1 MA Advantage, r2 Traditional Medicare, r3 Employer-sponsored, r4 Other.
- `XRANDOMPICK` (radio, hidden): MA vs ESI branch selector, set by quota balance logic (~2608–2633). r1=ESI, r2=MA.
- Quota tags (Decipher): `quo1` Total; `DemQuota` Dem; `quo2` GOP; `quo7` Block; `quo8/9/10/11` segment mins; `quo12` Typing module; `quo13` Wave One Equal Allocation; `quo_INSTYPE` Block_INSTYPE_Quota; ZIP dupes `quo3-6`; Age/Gender quotas; plus study-specific random quotas.
- Term variables: screenouts set via `vscreenout` + markers; terms include `Term_QS1`, `Term_QS2`, `term_QINSTYPE`, `Term_VOTE24`, `Term_PARTYID`, `Term_QPE2`, `Term_QPE3`.

## Typing Algorithm (1 paragraph)
The typing tool runs GOP and DEM scoring exec blocks (`algorithmRaw/algorithmCalculation`) to compute segment distances, then sets winners into `XGOP_SEG_FINAL_*` and `XDEM_SEG_FINAL_*`. Final assignment happens in `XSEG_ASSIGNED`: if GOP winner exists, it copies the GOP segment code; else if DEM winner exists it adds 10 to the DEM code. No term nodes sit between scoring and assignment; typing is guaranteed to complete when the earlier modules run. All three XMLs use the same 16-segment scheme and logic.

## Current Quota Structure
- All quotas read live from Decipher sheets via `<quota ... sheet="..." overquota="noqual">`; there are no hard-coded numeric caps in execs.
- Segment/wave/block: `Block Quota`, `Block_INSTYPE_Quota`, `Wave One Equal Allocation QUOTA`, segment minimums (`DEM First/Second Minimum`, `GOP First/Second Minimum`).
- Global/demographic: `Total Quota`, `Dem Quota`, `GOP Quota`, `Age Quota`, `Gender Quota`.
- Duplication control: `Dup_TWO/FIVE Quota` etc. via ZIP duplicate logic.
- Typing module: `TYPING MODULE Quota` (`quo12`).

## Current Branching (MA vs ESI)
```
QINSTYPE (+ QINS_MEDICARE if r7) -> XQINSTYPE (MA Adv=0, Trad=1, ESI=2, Other=3)
Read Block_INSTYPE_Quota for that XQINSTYPE row:
  ESI_BLOCK = limit-current of .../ESI_SEC
  MA_BLOCK  = limit-current of .../MA_SEC
if both >0: pick XRANDOMPICK via Block Quota cell markers (50/50)
if only one >0: force XRANDOMPICK to that open study (r1 ESI, r2 MA)
XRANDOMPICK.r1 -> ESI blocks (b1, ROI_ESI_FINAL)
XRANDOMPICK.r2 -> MA blocks (b3, ROI_MA_FINAL)
```
No eligibility guard beyond quotas; insurance type does not prevent cross-assign if quotas dictate.

## How to Verify Yourself (VS Code)
1) Variable map: `Ctrl+Shift+F` search `label="XSEG_ASSIGNED"`, `XQINSTYPE`, `XRANDOMPICK` in `PRISM_MA_ESI.xml`.
2) Typing completion: search `Final Segment Assignment` and scan up for `algorithmRaw`; confirm no `<term>` between scoring execs and `XSEG_ASSIGNED`.
3) Quotas: search `<quota label="` to list sheets; open the tag to read `sheet="..."`.
4) Branching: jump to lines ~2608–2633 to see the quota-based MA/ESI picker; then search `block label="b1" cond="XRANDOMPICK.r1"` and `block label="b3" cond="XRANDOMPICK.r2"`.
5) Term codes: search `<term ` to see all screenouts; search `<exit` near file top to see status URLs (`rst=1/2/3`) for Dynata and on-page messages for open sample.

## Routing Defaults (already discussed)
- Overquota: Implement soft-term with `ROUTER_STATUS=OVERQUOTA_TAGGED`, keep `XSEG_ASSIGNED`, optionally append `status=overquota_tagged&seg=` for panel; replace Dynata `rst=3` only if requested otherwise.
- Eligibility: Gate MA/ESI by insurance when both are open (MA stays MA, ESI stays ESI), while still honoring quota headroom and study priority; fallback tagging with `STUDY_INTENT` when both full.

## Ready-to-Send Client Update (concise)
- Confirmed: `XSEG_ASSIGNED` is the live segment output; MA/ESI routing is quota-driven via `XRANDOMPICK` and `Block_INSTYPE_Quota`; no `INS_*` flags exist today.
- Defaults (per our prior discussion): switch overquota to a polite soft-term that sets `ROUTER_STATUS=OVERQUOTA_TAGGED` and retains `XSEG_ASSIGNED`; gate MA vs ESI by insurance when both are open, with `STUDY_INTENT` tagging when full.
- Action: I’ll proceed with these defaults and stage for QA; tell me only if you prefer to keep the existing `rst=3` hard overquota or quota-only MA/ESI routing.
