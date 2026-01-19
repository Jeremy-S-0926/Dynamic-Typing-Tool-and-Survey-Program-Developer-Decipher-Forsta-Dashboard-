# Step 1c: Quota Sheet Status

**Files inspected:** `MA_ESI_quota.xls`, `GLP1-quota.xls` (Decipher live quota sheets).

**Findings (unblocked):**
- Real quota tables are in the `.xls` workbooks; they contain Total, Block, Block_INSTYPE_Quota, Wave One Equal Allocation, Dem/GOP minima, Age/Gender, random splits, and duplication controls.
- MA/ESI: `Total=*` (open); `Block_INSTYPE_Quota` caps by XQINSTYPE (r1 75/200, r2 75/200, r3 750/200, r4 300/200 ESI_SEC/MA_SEC); `Wave One Equal Allocation` caps each segment at `ESI=75`, `MA=50`; all other quotas show `inf` (Dem/GOP minima, Dem/GOP totals, Age/Gender, MA/ESI segment caps, random splits, dupes, typing module, block c1/c2).
- GLP1: `Total=*`; `Wave One Equal Allocation` lists numeric caps per segment (e.g., X0zv7 350, oiW1t 76, Bvr1C 100…); `Age Quota` ranges (18–29: 250-450; 30–44: 650-800; 45–54: 350-550; 55–64: 350-550; 65+: 600-850); `Gender Quota` ranges (Male 1100-1500; Female 1100-1500; Other 0-100; PNA 0); block c1/c2 and all segment minima/dup/random splits/typing module are `inf`.

**Deliverables:**
- `discovery/Quota_Map.md` documents sheet→tag mapping, numeric caps, segment IDs, and open questions.
- `Defines` tabs in both workbooks map quota IDs to questions/segments (used in the mapping doc).

**Open questions to confirm:**
- `*` and `inf` appear to mean “open/no cap”; confirm no separate cap source exists outside these sheets.
- GLP1 Age/Gender min-max strings—confirm if both bounds are enforced or if these are notes with upper bounds only.
- MA/ESI Block c1/c2 are uncapped; balance currently relies on Block_INSTYPE_Quota + Wave Equal caps. Should block caps be set for deterministic 50/50 when both sides are open?
- Provide AL/VAX/PULSE quota sheets if those studies will route through this router.

**Status:** Step 1c unblocked and documented; awaiting the clarifications above.
