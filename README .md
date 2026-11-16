# README

# Transportation Optimization for Emergency-Kit Distribution in Egypt

**Operations Research Project | Sphinx University | Group OR.9-11.A3**

A Transportation Problem (Linear Programming) that finds the cheapest way to deliver emergency kits from 4 manufacturers and 1 proposed Assiut hub to 8 hospitals in Egypt. The solution uses **Least Cost (LCM)**, **Vogel’s Approximation (VAM)** and **MODI**, and is checked with an independent LP solver.

---

## Key Results

| Method | Total cost (thousand EGP) | Gap to optimum |
| --- | --- | --- |
| Least Cost (LCM) | 23,130 | +2.99% |
| Vogel (VAM) | 22,658 | +0.89% |
| **MODI (optimal)** | **22,458** | 0% |
| LP solver (check) | 22,458 | same as MODI |
- MODI needs **2 pivots** starting from VAM. Starting from LCM it reaches the same optimum.
- **Hub analysis:** without the proposed Assiut hub the cost is 22,932 and 8 units are not delivered (Aswan). With the hub it is 22,458 and all 106 units are delivered, a saving of **474 thousand EGP** (transport cost only).

---

## The Problem

- **Sources (supply, units):** VACSERA 16, EIPICO 30, Pharco 22, Assiut Emergency Hub (proposed) 18, Amoun 30. Total = 116.
- **Hospitals (demand, units):** Alexandria 18, Mansoura 10, Assiut 24, Aswan 14, Hurghada 10, Sohag 12, Minya 10, Tanta 8. Total = 106.
- **Unit:** 1 unit = 100 emergency kits. **Cost:** thousand EGP per unit.
- Supply is bigger than demand by 10, so a **dummy destination** (“Unused Supply”, cost 0) balances the model.

**Cost rule used to build the data:**

```
Cost = round(2 + 0.8 x distance_km + 1.5 x relay_stops)
```

Road distance = 1.1 x straight-line distance, and one relay stop is added for every full 100 km.

---

## Mathematical Model

```
Minimize    Z = sum_i sum_j c_ij * x_ij
Subject to  sum_j x_ij = s_i     (each source ships its supply)
            sum_i x_ij = d_j     (each hospital gets its demand)
            x_ij >= 0
```

After balancing: 5 x 9 = 45 variables and 14 equality constraints (one redundant), so every plan has **13 basic cells** (m + n - 1).

---

## Methods

| Method | Idea |
| --- | --- |
| **LCM** | Pick the cheapest open cell, ship the maximum, close the row or column. |
| **VAM** | Penalty = (2nd cheapest cost) - (cheapest cost). Start with the largest penalty. |
| **MODI** | Solve `u_i + v_j = c_ij` on basic cells, compute `Delta_ij = c_ij - (u_i + v_j)` on empty cells. If all `Delta >= 0` the plan is optimal; otherwise move `theta` units around a closed loop (cost change = `Delta x theta`). |

---

## Repository Files

| File | Description |
| --- | --- |
| `2-OR_Assignment3_Transportation_Problem.ipynb` | Main notebook: solves the problem, verifies it, draws the charts. |
| `Project_Data.csv` | Data set: 40 rows (5 sources x 8 hospitals) with coordinates, supply, demand, distance, relay stops and cost. |
| `3-OR_Transportation_Report.docx` | Full project report. |
| `4-Emergency-Kit-Transportation-Optimization_edited.pptx` | Presentation slides. |
| `README.md` | This file. |

---

## Notebook Structure

| Part | Content |
| --- | --- |
| **1. Load and Balance** | Reads the CSV, builds supply, demand and the cost matrix, adds the dummy node. |
| **2. VAM and LCM** | Starting solutions, with a validity check for every plan. |
| **3. MODI and LP** | Optimality test and improvement loop, then an independent LP check. |
| **4. Results and Charts** | Allocation tables, 3 charts, hub analysis and a summary table. |

**Charts:** (1) total cost by method, (2) cost per used route, (3) share of shipped units per source. The dummy node is excluded from the charts.

---

## How to Run

**Requirements:** Python 3.9+ and the packages below.

```bash
pip install numpy pandas matplotlib scipy jupyter
```

1. Put `Project_Data.csv` in the same folder as the notebook.
2. Open the notebook in Jupyter, VS Code, or Google Colab (upload the CSV in Colab).
3. Run all cells from top to bottom.

The notebook raises an error if a check fails (supply, demand or negative shipments), so a clean run means the results are valid.

---

## Optimal Plan (MODI)

| From | To | Units | Unit cost | Route cost |
| --- | --- | --- | --- | --- |
| VACSERA | Assiut | 6 | 287 | 1,722 |
| VACSERA | Minya | 10 | 199 | 1,990 |
| EIPICO | Mansoura | 10 | 81 | 810 |
| EIPICO | Aswan | 4 | 628 | 2,512 |
| EIPICO | Hurghada | 10 | 354 | 3,540 |
| Pharco | Alexandria | 18 | 26 | 468 |
| Assiut Hub | Assiut | 18 | 10 | 180 |
| Amoun | Aswan | 10 | 626 | 6,260 |
| Amoun | Sohag | 12 | 368 | 4,416 |
| Amoun | Tanta | 8 | 70 | 560 |
| **Total** |  | **106** |  | **22,458** |

The 10 unused units stay at EIPICO (6) and Pharco (4). The two Aswan routes are about 39% of the total cost.

---

## Limitations

- Supply and demand are **team assumptions**, not official figures.
- Costs follow a distance-based formula, not real freight prices.
- The hub’s setup and stocking cost is **not** included, so the 474 thousand EGP saving is not a full investment case.
- The model is linear and ignores vehicle capacity, time windows and volume discounts.
- One product type and one delivery round only.
- The dummy node has cost 0, so a shortage has no penalty in the objective.

## Future Work

Add the hub’s setup cost, vehicle capacity limits, shortage penalties, real freight prices, and changing demand during an emergency.

---

## References

1. Course lectures 2 and 3 (Linear Programming and Transportation Models), Sphinx University.
2. H. A. Taha, *Operations Research: An Introduction*, Pearson.
3. F. S. Hillier and G. J. Lieberman, *Introduction to Operations Research*, McGraw-Hill.
4. N. V. Reinfeld and W. R. Vogel, *Mathematical Programming*, Prentice-Hall, 1958.