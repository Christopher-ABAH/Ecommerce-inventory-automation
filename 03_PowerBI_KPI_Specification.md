# Phase 4: Executive Power BI Dashboard Specification

**Project:** E-Commerce Order Management & Inventory Automation System  
**Deliverable:** Data Visualization Requirements & KPI Mapping Spec  
**Target Audience:** Operations Manager, Logistics Lead, Executive Stakeholders  

---

## 1. Dashboard Objective & Layout Architecture

The objective of this reporting layer is to provide real-time operational visibility into inventory health, order processing performance, and revenue trends. The dashboard is structured into three primary visual zones:

* **Executive Summary Header:** High-level scorecard KPIs refreshed in real time.
* **Operational Control Panel:** Interactive inventory status heatmaps and automated reorder alerts.
* **Fulfillment & Sales Analytics:** Daily order throughput and top-performing product categories.

---

## 2. KPI Data Mapping Matrix

| Visual / Tile | Metric Name | Source Table & Logic | Business Purpose | Target Threshold |
| :--- | :--- | :--- | :--- | :--- |
| **KPI Card 1** | Total Orders | `COUNT(Orders.OrderID)` where `OrderStatus != 'Cancelled'` | Track overall fulfillment volume | Daily target: 100+ orders |
| **KPI Card 2** | Net Revenue | `SUM(Orders.TotalAmount)` | Monitor gross throughput | MoM Growth: +10% |
| **KPI Card 3** | Low Stock Alerts | `COUNT(Products.ProductID)` where `StockQuantity <= ReorderPoint` | Immediate operational alert for supplier reordering | Maintain at 0 critical stockouts |
| **Bar Chart** | Top 10 Moving SKUs | `SUM(Order_Items.Quantity)` grouped by `ProductName` | Guide purchasing and warehouse floor planning | Top 20% inventory velocity |
| **Line Chart** | Revenue Trend | `SUM(TotalAmount)` grouped by `DATE(OrderDate)` | Track sales cadence across periods | Stable daily trajectory |

---

## 3. Data Refresh & Security Protocol

* **Data Refresh Frequency:** Scheduled DirectQuery / Gateway refresh every 15 minutes to capture automated webhook updates.
* **Row-Level Security (RLS):**
  * **Warehouse Operators:** Filtered view restricted to `Pending` and `Processing` order queues.
  * **Executives:** Unrestricted view across financial metrics, gross margin, and historical sales data.
