# Gett Failed Orders: Exploratory Analysis
**Python · Pandas · Matplotlib · H3 · Folium**

---

## Overview

Exploratory data analysis investigating matching metrics for orders that did not complete successfully on the Gett ride-hailing platform — i.e., the customer didn't end up getting a car. The analysis covers failure reasons, hourly trends, cancellation timing, ETA distribution, and geospatial patterns using hexagonal binning.

---

## Tech Stack

| Layer | Tool |
|---|---|
| Language | Python |
| Data Manipulation | Pandas |
| Visualization | Matplotlib |
| Geospatial | H3, Folium |
| Environment | Jupyter Notebook |

---

## Dataset

Two CSV files provided as part of the assignment:

- `data_orders.csv` — order-level data including timestamp, coordinates, ETA, order status, driver assignment, and cancellation time
- `data_offers.csv` — map of order numbers to offer IDs

**Order status mapping:**
- `4` — Cancelled by client
- `9` — Cancelled by system (rejected)

---

## Analysis Tasks

1. **Failure Distribution** — Breakdown of failed orders by reason: client cancellations before/after driver assignment, and system rejections. Identifies the highest-volume failure category.

2. **Hourly Failure Trends** — Distribution of failed orders by hour of day. Identifies peak failure hours and category-level patterns across the day.

3. **Cancellation Time by Hour** — Average time to cancellation with and without a driver assigned, plotted by hour. Outliers removed before analysis.

4. **ETA Distribution by Hour** — Average ETA across hours of the day and interpretation of demand/supply patterns.

5. **Geospatial Hexagon Map (Bonus)** — Using H3 resolution-8 hexagons, calculates how many hexes contain 80% of all orders and visualizes them on a Folium map colored by failure count.

---

## Key Findings

- Client cancellations without a driver assigned are the dominant failure category across all hours
- Failures peak during evening/night hours, aligning with higher demand periods
- Cancellation time with a driver assigned is consistently longer than without, suggesting customers wait longer when a driver is en route
- 80% of all orders are concentrated in a small number of geographic hexagons, revealing clear spatial clustering

---

## Files

| File | Description |
|---|---|
| `Gett_Analysis.ipynb` | Main Jupyter notebook with full analysis |
| `data_orders.csv` | Order-level raw data |
| `data_offers.csv` | Offer ID mapping |
| `datasets.zip` | Zipped raw data |
| `map.html` | Interactive Folium hexagon map |

---

## Author

**Talay Kamali** — Data Analyst  
[GitHub](https://github.com/talaygh)
