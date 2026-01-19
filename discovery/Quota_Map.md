# Quota Map (MA/ESI + GLP1)

Sources inspected (Jan 19, 2026):
- `MA_ESI_quota.xls` (Decipher live sheets used by `PRISM_MA_ESI.xml`).
- `GLP1-quota.xls` (Decipher live sheets used by `PRISM_GLP1`).

## Sheet → Tag Reference (Decipher)
- `Total Quota` → `quo1` (global cap).
- `Block Quota` → `quo7` (block picker c_1 / c_2).
- `Block_INSTYPE_Quota` → `quo_INSTYPE` (MA vs ESI by insurance type).
- `Wave One Equal Allocation QUOTA` → `quo13` (segment-level caps by study/wave).
- `Dem Quota` → `DemQuota`; `GOP Quota` → `quo2`.
- `DEM First/Second Minimum`, `GOP First/Second minimum` → segment minimums (`quo8/9/10/11`).
- `Age Quota`, `Gender Quota` → age/gender quota tags; `TYPING MODULE Quota` → `quo12`.
- `Dup_TWO/THREE/FOUR/FIVE Quota` → duplication controls (`quo3-6`).
- Random splits (`random Quota*`, `CRITIC1 Random Quota*`) feed study-specific branching checks.

## MA/ESI (MA_ESI_quota.xls)
- **Total Quota:** `Total=*` (open; no numeric cap present).
- **Block_INSTYPE_Quota (XRANDOMPICK by XQINSTYPE):**
  - XQINSTYPE_1 Medicare Advantage: `ESI_SEC=75`, `MA_SEC=200`.
  - XQINSTYPE_2 Traditional Medicare: `ESI_SEC=75`, `MA_SEC=200`.
  - XQINSTYPE_3 Employer-sponsored: `ESI_SEC=750`, `MA_SEC=200`.
  - XQINSTYPE_4 Other: `ESI_SEC=300`, `MA_SEC=200`.
- **Wave One Equal Allocation (segment caps, ESI vs MA):** each segment `ESI_SEC=75`, `MA_SEC=50`.
  - 1 Consumer Empowerment Champions (X0zv7)
  - 2 Holistic Health Naturalists (oiW1t)
  - 3 Traditional Conservatives (Bvr1C)
  - 4 Paleo Freedom Fighters (kUXHr)
  - 5 Price Populists (VpI7F)
  - 6 Wellness Evangelists (Ud4vK)
  - 7 Radical Futurists (OgFcq)
  - 8 Vaccine Skeptics (IxUJU)
  - 9 Medical Freedom Libertarians (WaHOc)
  - 10 Trust The Science Pragmatists (R06Sb)
  - 11 Idealists (Yjhg0)
  - 12 Progressives (Z8Vb5)
  - 13 Protectionists (o2mX2)
  - 14 Abundance (hyVPd)
  - 15 Incrementalists (t1ZJA)
  - 16 Institutionalists (ZpKT7)
- **Block Quota:** `c_1=inf`, `c_2=inf` (no cap; acts as marker for XRANDOMPICK even/odd).
- **Segment minimums:** all INF (`GOP First minimum`, `GOP Second minimum`, `DEM First Minimum`, `DEM Second Minimum`).
- **Dem/GOP totals:** `Dem Quota`, `GOP Quota` all INF.
- **Age/Gender:** all INF (IDs map via `Defines` to QAGECAT5 and QGENDER).
- **Study-specific segment caps:** `ESI Quota`, `MA Quota` rows 1–16 all INF.
- **Typing module:** `TYPING MODULE Quota` rows GOP/DEM both INF.
- **Random splits:** `random Quota` through `random Quota5` all INF (splits for statements, INS_EXPA/B, etc.).
- **Duplication controls:** `Dup_TWO/FIVE` etc. all INF (no active dedupe cap).

## GLP1 (GLP1-quota.xls)
- **Total Quota:** `Total=*` (open).
- **Wave One Equal Allocation (segment caps):**
  - 1 Consumer Empowerment Champions (X0zv7): `350`
  - 2 Holistic Health Naturalists (oiW1t): `76`
  - 3 Traditional Conservatives (Bvr1C): `100`
  - 4 Paleo Freedom Fighters (kUXHr): `81`
  - 5 Price Populists (VpI7F): `82`
  - 6 Wellness Evangelists (Ud4vK): `101`
  - 7 Radical Futurists (OgFcq): `30`
  - 8 Vaccine Skeptics (IxUJU): `81`
  - 9 Medical Freedom Libertarians (WaHOc): `350`
  - 10 Trust The Science Pragmatists (R06Sb): `75`
  - 11 Idealists (Yjhg0): `350`
  - 12 Progressives (Z8Vb5): `151`
  - 13 Protectionists (o2mX2): `250`
  - 14 Abundance (hyVPd): `98`
  - 15 Incrementalists (t1ZJA): `101`
  - 16 Institutionalists (ZpKT7): `350`
- **Age Quota (ranges shown in sheet):** 18–29: `250-450`; 30–44: `650-800`; 45–54: `350-550`; 55–64: `350-550`; 65+: `600-850`.
- **Gender Quota (ranges shown):** Male `1100-1500`; Female `1100-1500`; Other `0-100`; PNA `0`.
- **Block Quota:** `c_1=inf`, `c_2=inf`.
- **Segment minimums:** all INF (GOP/DEM first and second minima).
- **Dem/GOP totals:** all INF.
- **Random splits:** `CRITIC1 Random Quota` and `CRITIC1 Random Quota1` all INF (11A/B, 12A/B splits).
- **Typing module:** INF (GOP/DEM rows).
- **Duplication controls:** all INF.

## Open Questions / Assumptions
- `Total=*` and `inf` cells imply no cap; confirm if any hidden counts live elsewhere in Decipher.
- For MA/ESI, `Block Quota` c_1/c_2 are uncapped; XRANDOMPICK balance relies on Block_INSTYPE_Quota and Wave Equal caps. Confirm if deterministic 50/50 is desired or if block caps should be set.
- GLP1 Age/Gender ranges list min-max strings (e.g., `250-450`); verify whether Decipher interprets these as two-sided bounds or if these are notes and only the upper bound should be enforced.
- No counts present in sheets; Decipher will track completes. If external counts exist, we need the latest numbers before go-live.
- Other studies (AL_VAX, PULSE) not covered here; request their quota sheets if they will be routed.
