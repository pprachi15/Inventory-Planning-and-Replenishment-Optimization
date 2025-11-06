# 🏭 Supply Chain Inventory Planning & Replenishment Optimization  
**SQL · Forecasting · Supply Chain Analytics**

---

## 📌 Problem Statement
A fast-growing EV manufacturer is missing critical parts across a **10-warehouse network**, causing assembly line delays, emergency expedites, and excess capital tied up in the wrong SKUs. Inventory parameters are set manually and updated irregularly, so **reorder points (ROP), safety stock (SS), and order sizes** don’t reflect actual demand variability and lead-time risk.

## 💼 Business Problem We’re Solving
Build a data-driven **inventory planning system** that:
- Forecasts demand at **SKU × warehouse** level,
- Calculates statistically sound **Safety Stock** and **Reorder Points**, and
- Recommends **order quantities** that minimize total cost (holding + ordering + stockout/expedite),
while meeting a target **95% service level** for production-critical SKUs.

**Success looks like:** fewer stockouts/expedites, higher on-time part availability, and a measurable **reduction in holding costs (~15%)** with clear trade-offs for operations and finance.

## ❓ Five Key Questions We Answer
1. **What should the Safety Stock and Reorder Point be** for each SKU-warehouse pair given forecasted demand and stochastic lead time?  
2. **Which SKUs consume the most working capital** without improving service (candidates for SS/ROP tuning)?  
3. **What order quantity (EOQ or policy-based)** best balances ordering cost, holding cost, and supplier constraints (MOQ, review cadence)?  
4. **How do service-level targets (90% vs 95% vs 98%)** change total cost and stockout risk across the network?  
5. **Where is near-term stockout risk highest,** and what proactive orders should we place this week to prevent line stops?

---

## 🧱 Data Schema (simulated)
- `sku_master.csv`  
  `sku_id, sku_name, abc_class, unit_cost, holding_cost_pct, supplier_id`
- `demand_history.csv` (weekly)  
  `week_start, sku_id, warehouse_id, demand_units`
- `lead_times.csv` (days)  
  `sku_id, warehouse_id, lead_time_mean, lead_time_std, on_time_pct`

> Scale used in experiments: **200+ SKUs × 10 warehouses × 104 weeks (~100K rows).**

---

## 📐 Core Formulas
Let weekly demand ~ Normal(μ, σ). Let lead time in weeks = **L**.

- Demand during lead time:  
  - **μ_L = μ × L**  
  - **σ_L = √L × σ**
- Service level factor: **z** (e.g., 1.28 = 90%, 1.64 = 95%)
- **Safety Stock:** `SS = z × σ_L`  
- **Reorder Point:** `ROP = μ_L + SS`  
- **EOQ (Wilson):** `EOQ = √((2 × D × K) / H)`  
  - D = annual demand, K = order/setup cost, H = annual holding cost per unit

---

## 🧪 Approach (1-pager)
- Roll weekly demand to compute **μ (mean)** and **σ (std)** by **SKU × warehouse** (12-week window).  
- Join to **lead-time mean & std** → compute **σ during lead time** and **ROP = μ·L + z·σ·√L**.  
- Recommend **EOQ** and enforce policy: `order = max(EOQ, μ_L, MOQ)` on a weekly review.  
- Score impact vs baseline: **fill rate, expedites, holding $, working capital, turns**.  
- Publish KPI view; add scenarios for **z-score** and **lead-time shocks**.

---
## ✅ Expected Outcomes
- Holding cost ↓ ~15% via optimized reorder thresholds and cycle frequency.
- Service level ↑ (e.g., +6pp) by aligning SS/ROP with real demand variability.
- Fewer emergency expedites and better parts readiness for on-time builds.

---
## 🚀 Getting Started
- Drop CSVs into data/raw/ (see schemas).
- Run SQL scripts in sql/ in order 01 → 04 (coming soon).
- (Optional) Run src/compute_planning.py to export planning_table.csv (coming soon).
- Connect Tableau/Power BI to visualize KPIs and scenarios.
