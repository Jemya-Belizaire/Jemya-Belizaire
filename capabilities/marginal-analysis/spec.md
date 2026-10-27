---
type: spec
capability: marginal-analysis
engagement: perfect-competition
date: 2026-10-13
status: complete
---

# Marginal Analysis - Model Specification

## Purpose

This model will determine the most profitable allocation of 64 planting beds among tomatoes, carrots, and mesclun over a 36-week growing season. It will evaluate revenue, fertilizer, labor, diminishing returns, and resource constraints to identify the optimal planting combination and compare it with my original prediction of 18 tomato beds, 18 carrot beds, and 25 mesclun beds.

## Inputs - The Named Contract

Every input must have a named range, a defined unit, and a source. Derived inputs must be calculated using Excel formulas rather than entered as rounded values.

| Name | Value or Formula | Unit | Source |
|---|---|---|---|
| TOM_CAP | 20 | beds | Case crop table |
| CAR_CAP | 20 | beds | Case crop table |
| MES_CAP | 30 | beds | Case crop table |
| TOM_PRICE | 8800 | USD/bed | Case crop table |
| CAR_PRICE | 2094 | USD/bed | Case crop table |
| MES_PRICE | 2700 | USD/bed | Case crop table |
| TOM_HRS | 2.5 | hrs/week/bed | Case crop table |
| CAR_HRS | =2.5/3 | hrs/week/bed | Case crop table |
| MES_HRS | =2.5/2 | hrs/week/bed | Case crop table |
| TOM_FERT | 880 | USD/bed | Case crop table |
| CAR_FERT | 440 | USD/bed | Case crop table |
| MES_FERT | 880 | USD/bed | Case crop table |
| TOM_DIM | 0.10 | decimal | Case crop table |
| CAR_DIM | 0.025 | decimal | Case crop table |
| MES_DIM | 0.0125 | decimal | Case crop table |
| WEEKS | 36 | weeks | Case farm table |
| FIXED_COST | 20000 | USD/season | Case farm table |
| TOTAL_BEDS | 64 | beds | Case farm table |
| HOURS_PER_WEEK | 40 | hrs/week | Case farm table |
| OWNER_FIELD_SHARE | 0.50 | decimal | Case farm table |
| OWNER_ANNUAL_VALUE | 50000 | USD/season | Case farm table |
| TEMP_ANNUAL_VALUE | 25000 | USD/worker/season | Case farm table |
| MAX_TEMP_WORKERS | 4 | workers | Case farm table |
| OWNER_HRS | =HOURS_PER_WEEK*WEEKS*OWNER_FIELD_SHARE | hrs/season | Derived |
| OWNER_RATE | =OWNER_ANNUAL_VALUE/(HOURS_PER_WEEK*WEEKS) | USD/hr | Derived |
| TEMP_RATE | =TEMP_ANNUAL_VALUE/(HOURS_PER_WEEK*WEEKS) | USD/hr | Derived |
| TEMP_HRS_PER_WORKER | =HOURS_PER_WEEK*WEEKS | hrs/season/worker | Derived |
| TEMP_CAPACITY | =MAX_TEMP_WORKERS*TEMP_HRS_PER_WORKER | hrs/season | Derived |
| TOTAL_LABOR_CAPACITY | =OWNER_HRS+TEMP_CAPACITY | hrs/season | Derived |

### Decision Variables

The three planting decisions are:

- TOM_BEDS: Number of tomato beds
- CAR_BEDS: Number of carrot beds
- MES_BEDS: Number of mesclun beds

These are editable, nonnegative whole numbers, not fixed inputs.

All named inputs and derived values must retain their full calculation precision. Display rounding must not change the underlying values used in formulas.

## Structure

The workbook will contain five worksheets:

1. **Inputs:** All named inputs, derived input formulas, units, sources, and three editable planting decisions.

2. **Cost Structure:** Crop revenue, fertilizer costs, labor hours, owner and temporary labor allocation, blended labor rate, total costs, and seasonal profit.

3. **Marginal Cost:** Separate quantity schedules from zero to each crop's maximum, showing revenue, labor hours, fertilizer, standalone total variable cost, marginal cost, and price-minus-marginal-cost differences.

4. **Optimization:** Profit objective, three planting decisions linked to Inputs, Solver constraints, and clearly labeled constraint checks.

5. **Audit:** Acceptance criteria, formula-based PASS/FAIL checks, hand calculations, Solver starting-point results, and comparison with the Farm Profit Lab.

All calculated cells must contain Excel formulas rather than pasted values.

## Calculation Logic

### Crop Calculations

For each crop c with quantity q:

LABOR_HRS_c(q) = q * HRS_c * WEEKS * (1 + DIM_c)^q

REVENUE_c(q) = q * PRICE_c

FERT_COST_c(q) = q * FERT_c

TOTAL_LABOR_HRS = LABOR_HRS_TOM(TOM_BEDS) + LABOR_HRS_CAR(CAR_BEDS) + LABOR_HRS_MES(MES_BEDS)

### Labor Allocation

OWNER_HRS_USED = MIN(TOTAL_LABOR_HRS, OWNER_HRS)

TEMP_HRS_USED = MAX(0, TOTAL_LABOR_HRS - OWNER_HRS_USED)

OWNER_LABOR_COST = OWNER_HRS_USED * OWNER_RATE

TEMP_LABOR_COST = TEMP_HRS_USED * TEMP_RATE

TOTAL_LABOR_COST = OWNER_LABOR_COST + TEMP_LABOR_COST

TEMP_WORKERS_REQUIRED = TEMP_HRS_USED / TEMP_HRS_PER_WORKER

BLENDED_RATE = IF(TOTAL_LABOR_HRS = 0, 0, TOTAL_LABOR_COST / TOTAL_LABOR_HRS)

ALLOCATED_LABOR_COST_c = LABOR_HRS_c(q) * BLENDED_RATE

### Revenue, Costs, and Profit

TOTAL_REVENUE = SUM(REVENUE_c)

TOTAL_FERT_COST = SUM(FERT_COST_c)

TOTAL_OPERATING_COST = TOTAL_FERT_COST + TOTAL_LABOR_COST

SEASON_PROFIT = TOTAL_REVENUE - TOTAL_OPERATING_COST - FIXED_COST

The objective is to maximize SEASON_PROFIT while satisfying all planting and labor constraints.

### Standalone Marginal Cost

For each crop's standalone marginal-cost schedule, hold the other two crop quantities at zero.

STANDALONE_COST_c(q) = FERT_COST_c(q) + TOTAL_LABOR_COST evaluated using LABOR_HRS_c(q) only

MC_c(q) = STANDALONE_COST_c(q) - STANDALONE_COST_c(q - 1), for q >= 1

PRICE_MINUS_MC_c(q) = PRICE_c - MC_c(q)

For q = 0:

- Labor hours equal zero.
- Revenue equals zero.
- Fertilizer and variable costs equal zero.
- Marginal cost is not applicable.

The marginal-cost schedules must use formulas linked to the named inputs.

Standalone marginal costs must be calculated independently for each crop rather than using the farm's optimized crop mix.

## Conventions

Owner labor is used before temporary labor.

The owner supplies up to 720 field hours per season. The owner's hourly rate is calculated from the original case figures rather than entered as a rounded constant.

Temporary labor covers the remaining labor requirement using the derived temporary hourly rate.

Labor costs are allocated across crops using the farm-wide blended hourly rate. Do not assign the owner rate to one crop and the temporary rate to another.

The maximum temporary labor capacity is:

MAX_TEMP_WORKERS * TEMP_HRS_PER_WORKER = 5760 hours

Total field labor capacity is:

OWNER_HRS + TEMP_CAPACITY = 6480 hours

All planting decisions must be whole numbers between zero and their individual crop caps.

Combined planting quantity cannot exceed 64 beds.

Keep full calculation precision internally. Round displayed dollar values to the nearest dollar for summary reporting without rounding intermediate formulas.

Marginal cost may fall or rise depending on quantity and the labor-rate transition. Do not force the marginal-cost schedule to increase monotonically.

## Optimization Requirements

The Solver model must maximize SEASON_PROFIT.

### Changing Cells

- TOM_BEDS
- CAR_BEDS
- MES_BEDS

### Constraints

- TOM_BEDS >= 0
- CAR_BEDS >= 0
- MES_BEDS >= 0
- TOM_BEDS <= TOM_CAP
- CAR_BEDS <= CAR_CAP
- MES_BEDS <= MES_CAP
- TOM_BEDS + CAR_BEDS + MES_BEDS <= TOTAL_BEDS
- TEMP_HRS_USED <= TEMP_CAPACITY
- TOM_BEDS, CAR_BEDS, and MES_BEDS must be integers.

### Solver Method

Use GRG Nonlinear with integer decision constraints.

Run Solver from two starting points:

1. 0 tomato beds, 0 carrot beds, 0 mesclun beds.
2. 20 tomato beds, 0 carrot beds, 0 mesclun beds.

Record the final allocation, profit, and Solver status for each run.

Compare whether both starting points reach the same solution.

The final workbook must retain the selected Solver solution and, where supported, the saved Solver model.

## Validation Rules

The workbook must calculate acceptance checks using formulas rather than manually entered PASS/FAIL results.

### Acceptance Criteria

| Validation | Expected Result | Tolerance |
|---|---|---|
| One tomato bed labor | 99 hours | +/- 0.01 hours |
| Optimal tomato beds | 10 | Exact |
| Optimal carrot beds | 20 | Exact |
| Optimal mesclun beds | 30 | Exact |
| Total planted beds | 60 | Exact |
| Seasonal profit | $42,762 | +/- $5 |
| Tomato standalone P/MC crossing | Near 10 beds | +/- 1 bed |
| Carrot standalone P/MC crossing | Near 10 beds | +/- 1 bed |
| Mesclun standalone P/MC crossing | Near 6 beds | +/- 1 bed |

### Numerical Validation

For each numerical check:

ABS(ACTUAL_VALUE - EXPECTED_VALUE) <= TOLERANCE

The Audit worksheet must return PASS or FAIL using Excel formulas.

For example:

=IF(ABS(ACTUAL_VALUE-EXPECTED_VALUE)<=TOLERANCE,"PASS","FAIL")

For exact quantity checks, the tolerance is zero.

### Hand Calculation

For one tomato bed:

LABOR_HRS_TOM(1) = 1 * 2.5 * 36 * (1.10)^1

Expected result: 99 hours.

The workbook must independently calculate this value and verify that the difference is no greater than 0.01 hours.

### Marginal-Cost Validation

The standalone price and marginal-cost crossing points should occur near:

- Tomatoes: 10 beds
- Carrots: 10 beds
- Mesclun: 6 beds

The workbook must calculate marginal costs and price-minus-marginal-cost differences for every applicable quantity.

Identify the crossing using the change in sign of PRICE_MINUS_MC where applicable, rather than assuming the smallest absolute difference always represents a crossing.

Report the quantity associated with the crossing and compare it against the expected range.

The Audit worksheet must show the calculated result and whether it passes the tolerance.

### Constraint Validation

The workbook must calculate checks confirming that:

1. Each crop quantity is nonnegative.
2. Each crop quantity is an integer.
3. Each crop quantity remains within its individual cap.
4. Total planted beds do not exceed 64.
5. Temporary labor hours do not exceed 5760.
6. Total labor hours do not exceed 6480.
7. Temporary workers required do not exceed four.

Each constraint must display PASS or FAIL.

### Formula Validation

Confirm that:

- All original and derived inputs have named ranges.
- All calculated cells contain formulas.
- No intermediate calculations are rounded.
- No spreadsheet errors appear.
- Crop calculations link to named inputs.
- Optimization results link to the calculation worksheets.
- Audit checks update when the underlying values change.

The workbook must not use pasted answers in place of calculated results.

## Audit Testing

The Audit worksheet must document the following tests.

### Test 1: Hand Calculation

Calculate labor hours for one tomato bed.

Expected result: 99 hours.

Confirm that the calculated value is within 0.01 hours.

### Test 2: Optimal Allocation

Compare the Solver solution with the expected allocation:

- 10 tomato beds
- 20 carrot beds
- 30 mesclun beds
- 60 total beds

All quantity checks require exact matches.

### Test 3: Profit

Compare calculated seasonal profit against $42,762.

The result passes if the absolute difference is no greater than $5.

### Test 4: Solver Starting Points

Run Solver from:

- 0/0/0
- 20/0/0

Record each final allocation and seasonal profit.

State whether both runs converge to the same solution.

### Test 5: Intentional Validation Failure

Temporarily change an input or decision value to trigger a failed acceptance check.

Confirm that the corresponding validation formula changes from PASS to FAIL.

Restore the original value and confirm that the validation result updates correctly.

Record the test without leaving incorrect values in the final workbook.

### Test 6: Tomato Marginal-Cost Dip

Inspect the tomato marginal-cost schedule near six beds.

Identify and record the location of the dip.

Do not explain the cause at this stage.

### Test 7: Formula Integrity

Confirm that calculated results update when inputs change.

Verify that named ranges, formulas, and references remain intact.

Check for spreadsheet errors and record any corrections.

## Outputs

The workbook must report:

- Recommended beds for tomatoes, carrots, and mesclun
- Total beds planted and unused beds
- Revenue and fertilizer costs by crop
- Labor hours by crop and total labor hours
- Owner hours used and temporary hours used
- Temporary workers required
- Owner labor cost and temporary labor cost
- Blended hourly labor rate
- Total revenue
- Total operating costs
- Fixed costs
- Seasonal profit
- Standalone marginal-cost schedules for all three crops
- Price-minus-marginal-cost comparisons
- Solver solution and constraint status
- Validation results and audit findings
- Comparison with the original 18/18/25 prediction


## Audit Findings

### 1. Initial Allocation Audit

The original allocation of 18 tomato beds, 18 carrot beds, and 25 mesclun beds should have triggered the total labor capacity and temporary labor capacity constraint checks. The allocation required approximately 11,384 total labor hours, exceeding the 6,480-hour limit by 4,904 hours. It also required approximately 10,664 temporary labor hours, exceeding the 5,760-hour limit. Both checks should return FAIL for this allocation. The original allocation met the total bed capacity requirement but was infeasible because of insufficient labor capacity.

### 2. Solver Results

Two Solver runs were performed using different starting allocations.

| Item | Run 1 | Run 2 |
|---|---|---|
| Starting allocation | 0 / 0 / 0 | 20 / 0 / 0 |
| Tomato beds returned | 10.0040496 | 10.00404901 |
| Carrot beds returned | 20 | 20 |
| Mesclun beds returned | 30 | 30 |
| Integer constraint | FAIL | FAIL |

Solver returned nearly identical results for both starting allocations. Although Solver indicated that the constraints were satisfied, the tomato allocation was fractional.

Because the model requires whole-number bed allocations, neither Solver result fully satisfied the integer requirement.

### 3. Final Allocation Verification

The final allocation was manually verified using whole-number bed values.

| Crop | Final Beds |
|---|---:|
| Tomatoes | 10 |
| Carrots | 20 |
| Mesclun | 30 |
| **Total** | **60** |

The final allocation produced the following results:

| Audit Check | Result | Status |
|---|---|---|
| Season profit | $42,761.66 | PASS |
| Total beds | 60 / 64 | PASS |
| Total labor hours | 5,277.22 / 6,480 | PASS |
| Temporary labor hours | 4,557.22 / 5,760 | PASS |
| Whole-number bed allocation | 10 / 20 / 30 | PASS |
| Profit within $5 of check figure | Yes | PASS |

### 4. Conclusion

The final allocation of 10 tomato beds, 20 carrot beds, and 30 mesclun beds satisfies the bed capacity, labor capacity, temporary labor, and integer requirements.

The resulting season profit is approximately **$42,761.66**, matching the expected check figure within the required $5 tolerance.

Both Solver runs produced nearly identical fractional solutions despite the integer constraint being included. The final whole-number allocation was therefore manually verified rather than confirmed as an integer solution returned by Solver.

The workbook calculations and final allocation pass the feasibility and profit checks. The remaining limitation is Solver's failure to enforce the integer constraint during optimization.

The final workbook must preserve all formulas, named ranges, validation checks, and Solver results.

