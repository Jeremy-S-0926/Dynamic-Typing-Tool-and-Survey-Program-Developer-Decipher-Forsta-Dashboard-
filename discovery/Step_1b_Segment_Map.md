# Step 1b: Segment Definitions and Typing Algorithm

## Segment List (16, stable across MA/ESI, GLP1, AL_VAX)
1. Consumer Empowerment Champions
2. Holistic Health Naturalists
3. Traditional Conservatives
4. Paleo Freedom Fighters
5. Price Populists
6. Wellness Evangelists
7. Radical Futurists
8. Vaccine Skeptics
9. Medical Freedom Libertarians
10. Trust The Science Pragmatists
11. Idealists
12. Progressives
13. Protectionists
14. Abundance
15. Incrementalists
16. Institutionalists

## Scoring Logic (all three XMLs)
- Inputs: standardized z-scores stored in `XGOP_SEG.c4` (10 GOP dims) and `XDEM_SEG.c4` (10 DEM dims).
- GOP scoring: `algorithmRaw` computes squared-distance to a 10×10 centroid matrix; distances -> `XGOP_RAW` rows 1–10.
- DEM scoring: `algorithmRaw` computes squared-distance to a 10×6 centroid matrix; distances -> `XDEM_RAW` rows 1–6.
- Winners: take the lowest distance (`X*_SEG_FINAL_1`) and second-lowest (`X*_SEG_FINAL_2`) by sorting the `X*_RAW` values ascending.
- Final assignment: if `XGOP_SEG_FINAL_1` exists, `XSEG_ASSIGNED = GOP winner`; else if `XDEM_SEG_FINAL_1` exists, `XSEG_ASSIGNED = DEM winner + 10`.
- Weights: none beyond the embedded centroid matrices; the distance is unweighted sum of squared deltas against each centroid column.
- Tie-break: implicit—sorting the distance list; if equal values occur, first matching row keeps the slot (deterministic order of the list).

## Stability Check
- All three XMLs share the identical `XSEG_ASSIGNED` rows and the same GOP/DEM centroid matrices and logic.

## How to Verify in VS Code
1) Search `label="XSEG_ASSIGNED"` in each XML to confirm the identical 16-row list and final assignment logic.
2) Search `label="XGOP_RAW"` and `label="XDEM_RAW"` to see the centroid matrices and distance calculations.
3) Search `XGOP_SEG_FINAL_1` and `XDEM_SEG_FINAL_1` to see winners chosen as the smallest distances.
4) Confirm no `<term>` between scoring execs and `XSEG_ASSIGNED` (typing always completes).

## Example Pseudocode (matches XML)
```
# compute z inputs already in XGOP_SEG.c4 / XDEM_SEG.c4
for segment in GOP_SEGMENTS:
    distance = sum((resp - centroid_val)**2 for resp, centroid_val in zip(resp_answers, centroid_column))
    store distance in XGOP_RAW[segment]
GOP_winner = argmin(XGOP_RAW)

for segment in DEM_SEGMENTS:
    distance = sum((resp - centroid_val)**2 for resp, centroid_val in zip(resp_answers, centroid_column))
    store distance in XDEM_RAW[segment]
DEM_winner = argmin(XDEM_RAW)

if GOP_winner exists:
    XSEG_ASSIGNED = GOP_winner (1-10)
elif DEM_winner exists:
    XSEG_ASSIGNED = DEM_winner + 10 (11-16)
```
