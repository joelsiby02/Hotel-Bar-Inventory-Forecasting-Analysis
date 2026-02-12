# Hotel Bar Inventory Forecasting & Par Level Recommendation System

## Project Overview
This project builds an end-to-end demand forecasting and inventory recommendation system for a hotel chain operating multiple bars across locations.

The business problem is:
- **Stockouts** of high-demand items reduce guest satisfaction and revenue.
- **Overstock** of slow-moving items increases holding costs and waste.

The goal is to forecast demand and recommend inventory targets (Par levels) and reorder points (ROP) for each bar and brand, and simulate how the system behaves in real operations.

---

## Dataset
The dataset contains historical inventory movement logs with the following key columns:

- `Date Time Served`
- `Bar Name`
- `Alcohol Type`
- `Brand Name`
- `Opening Balance (ml)`
- `Purchase (ml)`
- `Consumed (ml)`  ← **treated as demand**
- `Closing Balance (ml)`

---

## Workflow Summary (End-to-End)

### 1) Data Preparation
- Converted the timestamp column into proper datetime.
- Aggregated transaction-level consumption into **daily demand**:
  - `date × bar × alcohol_type × brand → total consumed (ml)`
- Created a complete daily time series for each real bar-brand combination.
  - Missing days were filled with `0` demand (intermittent demand handling).

---

### 2) Demand Forecasting
Two baseline forecasting methods were tested:

1. **MA7 (Rolling 7-day Moving Average)**
2. **Seasonal Naive (7-day lag)**

Evaluation method:
- Time-based split (80% train / 20% test)
- MAE (Mean Absolute Error)

Result:
- Across 96 time series (6 bars × 16 brands), **MA7 outperformed seasonal naive in 69 cases (~72%)**.

So MA7 was chosen as the default forecasting approach.

---

### 3) Inventory Recommendation (ROP + Par Level)
Forecasts were converted into inventory decisions using standard inventory logic:

- **ROP (Reorder Point)** → when to place an order  
- **Par Level** → target inventory level after replenishment  

Assumptions:
- Lead time = **2 days**
- Service level = **95%** (z = 1.65)

Safety stock was calculated using demand volatility (historical standard deviation).

---

### 4) Simulation (How the system works in practice)
A day-to-day simulation was created to mimic real hotel bar operations:

- Inventory decreases daily based on demand.
- When inventory falls below ROP → an order is placed.
- Stock arrives after lead time.
- Orders are rounded to **750ml bottles** (real-world ordering constraint).
- KPIs tracked:
  - Stockout days
  - Number of orders
  - Inventory behavior over time

Example (Brown’s Bar – Yellow Tail):
- Orders placed: 39
- Stockout days: 5

---

## Key Outputs
For each `Bar × Alcohol Type × Brand`, the system produces:

- Daily demand forecast (MA7)
- Reorder Point (ROP)
- Par Level recommendation
- Simulation KPIs (stockouts, order frequency)

---

## Assumptions
- Demand = `Consumed (ml)`
- Missing days imply 0 demand
- Lead time assumed constant at 2 days (not provided in dataset)
- Service level fixed at 95%
- Ordering constraint: 750ml bottle rounding
- Normal operating conditions (no strikes/closures/events)

---

## How to Run
1. Open the notebook.
2. Run cells top to bottom.
3. Outputs include:
   - data exploration
   - forecasting comparison
   - par/rop table
   - simulation plots and KPIs

---

## Possible Improvements
- Use item-specific service levels (higher for fast movers, lower for slow movers)
- Add external demand drivers (weekdays, holidays, hotel occupancy, events)
- Learn real lead times per supplier/location
- Improve order sizing to reduce overstock from bottle rounding
- Track production metrics and retrain periodically

---
