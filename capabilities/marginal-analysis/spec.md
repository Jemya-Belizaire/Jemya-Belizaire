
---
type: spec
capability: marginal-analysis
engagement: perfect-competition
date: 2026-10-13
status: draft
---

# Marginal Analysis — Model Specification

## Purpose

This model will determine the most profitable allocation of 64 planting beds among tomatoes, carrots, and mesclun over a 36-week growing season. It will evaluate revenue, fertilizer, labor, diminishing returns, and resource constraints to identify the optimal planting combination and compare it with my original 18/18/25 prediction.

## Inputs — the named contract

| Name | Value | Unit | Source |
|---|---:|---|---|
| TOM_CAP | 20 | beds | Case crop table |
| CAR_CAP | 20 | beds | Case crop table |
| MES_CAP | 30 | beds | Case crop table |
| TOM_PRICE | 8800 | USD/bed | Case crop table |
| CAR_PRICE | 2094 | USD/bed | Case crop table |
| MES_PRICE | 2700 | USD/bed | Case crop table |
| TOM_HRS | 2.5 | hrs/week/bed | Case crop table |
| CAR_HRS | 0.833 | hrs/week/bed | Case crop table |
| MES_HRS | 1.25 | hrs/week/bed | Case crop table |
| TOM_FERT | 880 | USD/bed | Case crop table |
| CAR_FERT | 440 | USD/bed | Case crop table |
| MES_FERT | 880 | USD/bed | Case crop table |
| TOM_DIM | 0.10 | decimal | Case crop table |
| CAR_DIM | 0.025 | decimal | Case crop table |
| MES_DIM | 0.0125 | decimal | Case crop table |
| WEEKS | 36 | weeks | Case farm table |
| FIXED_COST | 20000 | USD/season | Case farm table |
| TOTAL_BEDS | 64 | beds | Case farm table |
| OWNER_HRS | 720 | hrs/season | Case farm table |
| OWNER_RATE | 34.72 | USD/hr | Case farm table |
| TEMP_RATE | 17.36 | USD/hr | Case farm table |
| TEMP_HRS_PER_WORKER | 1440 | hrs/season | Case farm table |
| MAX_TEMP_WORKERS | 4 | workers | Case farm table |

Decision variables: TOM_BEDS, CAR_BEDS, MES_BEDS. These are nonnegative whole numbers, not fixed inputs.

## Structure

The workbook will contain five worksheets:

1. **Inputs:** All named inputs and three editable planting decisions.
2. **Cost Structure:** Crop revenue, fertilizer costs, labor hours, owner/temporary labor allocation, blended labor rate, total costs, and seasonal profit.
3. **Marginal Cost:** Separate quantity schedules from zero to each crop's maximum, showing revenue, labor hours, fertilizer, marginal cost, and price-minus-marginal-cost differences.
4. **Optimization:** Profit objective, the three planting decisions linked to Inputs, Solver constraints, and clearly labeled constraint checks.
5. **Audit:** Acceptance criteria, formula checks, hand calculations, Solver starting-point results, and comparison with the Farm Profit Lab.

## Calculation logic

For each crop c with quantity q:

LABOR_HRS_c(q) = q * HRS_c * WEEKS * (1 + DIM_c)^q

REVENUE_c(q) = q * PRICE_c

FERT_COST_c(q) = q * FERT_c

TOTAL_LABOR_HRS = LABOR_HRS_TOM(TOM_BEDS) + LABOR_HRS_CAR(CAR_BEDS) + LABOR_HRS_MES(MES_BEDS)

OWNER_HRS_USED = MIN(TOTAL_LABOR_HRS, OWNER_HRS)

TEMP_HRS_USED = MAX(0, TOTAL_LABOR_HRS - OWNER_HRS_USED)

OWNER_LABOR_COST = OWNER_HRS_USED * OWNER_RATE

TEMP_LABOR_COST = TEMP_HRS_USED * TEMP_RATE

TOTAL_LABOR_COST = OWNER_LABOR_COST + TEMP_LABOR_COST

BLENDED_RATE = IF(TOTAL_LABOR_HRS = 0, 0, TOTAL_LABOR_COST / TOTAL_LABOR_HRS)

ALLOCATED_LABOR_COST_c = LABOR_HRS_c(q) * BLENDED_RATE

TOTAL_REVENUE = SUM(REVENUE_c)

TOTAL_FERT_COST = SUM(FERT_COST_c)

SEASON_PROFIT = TOTAL_REVENUE - TOTAL_FERT_COST - TOTAL_LABOR_COST - FIXED_COST

For each crop's standalone marginal-cost schedule, hold the other crop quantities at zero and calculate:

STANDALONE_COST_c(q) = FERT_COST_c(q) + TOTAL_LABOR_COST evaluated using LABOR_HRS_c(q) only

MC_c(q) = STANDALONE_COST_c(q) - STANDALONE_COST_c(q - 1), for q >= 1

PRICE_MINUS_MC_c(q) = PRICE_c - MC_c(q)

For q = 0, show labor and variable costs as zero and marginal cost as not applicable.

## Conventions

Owner labor is used before temporary labor. The owner supplies up to 720 hours at $34.72 per hour, and temporary labor covers the remaining requirement at $17.36 per hour.

Labor costs are allocated across crops using the farm-wide blended hourly rate. Do not assign the owner rate to one crop and the temporary rate to another.

The maximum temporary labor capacity is 4 * 1440 = 5760 hours. Total field labor cannot exceed 6480 hours.

All planting decisions must be whole numbers between zero and their individual crop caps. The combined planting quantity cannot exceed 64 beds.

Keep full calculation precision internally. Round displayed dollar values to the nearest dollar for summary reporting, without rounding intermediate formulas.

Marginal cost may fall or rise depending on quantity and the labor-rate transition. Do not force the marginal-cost schedule to increase monotonically.

## Validation rules

The workbook must pass these acceptance criteria:

1. For one tomato bed, labor must equal 1 * 2.5 * 36 * (1.10)^1 = 99 hours.
2. The expected optimal allocation is 10 tomato beds, 20 carrot beds, and 30 mesclun beds, using 60 total beds.
3. Expected seasonal profit is approximately $42,762.
4. Standalone price and marginal-cost crossing points should occur near 10 tomato beds, 10 carrot beds, and 6 mesclun beds.
5. All crop quantities must remain within their caps, combined beds must not exceed 64, and temporary workers must not exceed four.
6. Run Solver from both 0/0/0 and 20/0/0 using GRG Nonlinear with integer decision constraints. Record whether both runs reach the same solution.
7. Compare at least one intermediate marginal-cost value with the Farm Profit Lab.
8. Confirm that all calculated cells contain formulas, not pasted answers, and no spreadsheet errors appear.
9. Identify the tomato marginal-cost dip near six beds and record its location without explaining the cause yet.

## Outputs

The workbook must report:

- Recommended beds for tomatoes, carrots, and mesclun
- Total beds planted and unused beds
- Revenue and fertilizer costs by crop
- Labor hours by crop and total labor hours
- Owner hours used, temporary hours used, and temporary workers required
- Owner labor cost, temporary labor cost, and blended hourly rate
- Total revenue, total operating costs, fixed costs, and seasonal profit
- Standalone marginal-cost schedules and price comparisons for all three crops
- Solver solution and constraint status
- Validation results and audit findings

## Audit findings

To be completed after the workbook is generated and independently checked. Each finding will document what was checked, what was found, what error the check could have caught, and any corrective action taken.
