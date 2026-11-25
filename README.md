# Basel IRRBB Standardized Engine

EVE / NII measurement under the Basel IRRBB standardized framework, with
sensitivity to non-maturity-deposit (NMD) assumptions. Built step by step
from Lubinska, *Interest Rate Risk in the Banking Book* (Wiley, 2021),
cross-checked against the Basel standard.

## Status

| Step | Component | Status |
|---|---|---|
| 1 | Conventions, 19-bucket grid, position slotting | planned |
| 2 | OIS curve at bucket midpoints, six Basel shocks | planned |
| 3 | ΔEVE engine (contractual view), PV01 cross-check | planned |
| 4 | ΔNII engine (12M / 24M horizons) | planned |
| 5 | NMD behaviouralisation (stable / core / average life, caps) | planned |
| 6 | Deposit-assumption sensitivity (behavioural vs contractual) | planned |
| 7 | Optional: automatic options (KAO), dashboard, outlier test | planned |

## Scope and conventions

- Static balance sheet: accrual basis, book value = par.
- NII: constant balance sheet. EVE: run-off.
- Positions are grouped by repricing bucket (no instrument-level valuation);
  fixed-rate securities are mapped as par zero-coupons.
- Cash flows are notional repricing cash flows, slotted into 19 buckets.
- Signs: assets positive, liabilities negative.
- ΔEVE = shocked EVE − base EVE (positive = gain, negative = loss).
- Bucket edges: calendar-month, upper-inclusive (BCBS 368 Table 1:
  e.g. 1M < t ≤ 3M). The standard does not prescribe a day count.
- Run per material currency.

## Time buckets

19 buckets with midpoints (years), per BCBS 368, Table 1 (p. 24) and
book App. 2 §I (p. 221): 0.0028, 0.0417, 0.1667, 0.375, 0.625, 0.875, 1.25,
1.75, 2.5, 3.5, 4.5, 5.5, 6.5, 7.5, 8.5, 9.5, 12.5, 17.5, 25.

## The six Basel shocks

Parallel up, parallel down, steepener, flattener, short up, short down,
applied at each of the 19 midpoints (6 x 19 shifts per currency).

- Shock sizes (parallel / short / long) are inputs, not hard-coded.
- The book's sizes are the 2016 calibration. Basel recalibrated them in
  July 2024 (SRP31, effective 1 Jan 2026). Both vintages should be runnable.
- The current standard applies a floor, caps and rounding to shocked rates;
  the book's model does not.

## Design principles

- Pure functions for the maths; frozen configuration objects for calibration.
- Vectorised (NumPy); pandas only for labelled output.
- Shocks are independent of deposit assumptions, so Step 6 attributes
  changes cleanly.
- Golden tests against the book's tables; every function cites its source.

## Sources

- BCBS 368, *Interest rate risk in the banking book* (April 2016).
- Basel Framework SRP31 (effective 1 Jan 2026) and SRP98.
- BCBS, *Recalibration of shocks in the IRRBB standard* (d578, July 2024).
- BCBS consultation on additional Pillar 2 guidance for IRRBB
  (9 Nov 2026; not final).
- Lubinska (2021): Appendix 2 (model manual), Table 1.3 (p. 25),
  Table 4.1 (p. 119).

## Known discrepancies in the source book

- Undrawn credit lines: included (p. 212) vs excluded (p. 230).
- Example (c), p. 230: the third amortisation instalment (2024) is printed
  as 4-5Y; under calendar-month edges it falls in 3-4Y. Result depends on
  day count; documented as a test note.
- Core-deposit caps on p. 223 are the bank's own 50% haircut (35% / 25%),
  not the Basel caps (Table 4.1, p. 119: 90% / 70% / 50%).
- NII shock sizes: App. 1 (p. 205) says USD ±250 bp; Table 1.3 and
  App. 2 (p. 215) imply GBP ±250 bp / USD ±200 bp.

## Testing

Per step: unit tests plus the book's worked examples as fixtures.
Step 2 golden values: book Table 1.3 (p. 25).

## Disclaimer

Educational / research implementation. Not a regulatory submission.