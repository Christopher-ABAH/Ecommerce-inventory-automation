```markdown
# Phase 3: Relational Database Schema & SQL Analytics Pipeline

**Project:** E-Commerce Order Management & Inventory Automation System  
**Deliverable:** Database Schema Design (DDL) and Analytics SQL Queries (DML)  
**Role:** Technical Business Analyst  

---

## 1. Entity-Relationship (ER) Schema Design

To replace manual spreadsheet tracking, the automated system relies on four normalized relational tables designed to support transactional processing (OLTP) and Power BI executive reporting (OLAP).

* **`Customers`**: Stores customer identity and contact information.
* **`Products`**: Holds product metadata, pricing, current stock levels, and safety reorder thresholds.
* **`Orders`**: Captures top-level order headers, status lifecycle, and timestamps.
* **`Order_Items`**: Line-item detail table linking products to specific orders (junction table).

---

## 2. Database Definition Language (DDL) Script

```sql
-- Create Products Table with Automated Reorder Threshold Constraints
CREATE TABLE Products (
    ProductID INT PRIMARY KEY AUTO_INCREMENT,
    SKU VARCHAR(50) UNIQUE NOT NULL,
    ProductName VARCHAR(100) NOT NULL,
    UnitPrice DECIMAL(10,2) NOT NULL,
    StockQuantity INT NOT NULL DEFAULT 0,
    ReorderPoint INT NOT NULL DEFAULT 10,
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create Customers Table
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY AUTO_INCREMENT,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Email VARCHAR(100) UNIQUE NOT NULL,
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create Orders Table
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY AUTO_INCREMENT,
    CustomerID INT NOT NULL,
    OrderDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    OrderStatus VARCHAR(30) DEFAULT 'Pending', -- Pending, Processing, Shipped, Cancelled
    TotalAmount DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

-- Create Order_Items Junction Table
CREATE TABLE Order_Items (
    OrderItemID INT PRIMARY KEY AUTO_INCREMENT,
    OrderID INT NOT NULL,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL CHECK (Quantity > 0),
    UnitPrice DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);

```

---

## 3. SQL Analytics Queries for Power BI & Automation

### Query 1: Automated Low-Stock Alert Engine (FR-03 Trigger)

*Business Logic:* Identifies SKUs where current inventory is at or below the safety reorder threshold so automated supplier PO notifications can be triggered.

```sql
SELECT 
    ProductID,
    SKU,
    ProductName,
    StockQuantity,
    ReorderPoint,
    (ReorderPoint - StockQuantity) AS SuggestedReorderAmount
FROM Products
WHERE StockQuantity <= ReorderPoint
ORDER BY StockQuantity ASC;

```

---

### Query 2: Daily Order Fulfillment & Revenue Performance (Executive KPI)

*Business Logic:* Consolidates order throughput and total revenue generated per day for integration into the Power BI executive dashboard.

```sql
SELECT 
    DATE(OrderDate) AS ReportDate,
    COUNT(DISTINCT OrderID) AS TotalOrdersProcessed,
    SUM(TotalAmount) AS GrossRevenue,
    ROUND(AVG(TotalAmount), 2) AS AverageOrderValue
FROM Orders
WHERE OrderStatus IN ('Processing', 'Shipped')
GROUP BY DATE(OrderDate)
ORDER BY ReportDate DESC;

```

---

### Query 3: Top Selling SKUs and Inventory Turnover

*Business Logic:* Identifies fast-moving inventory line items to assist the Operations team with stock forecasting and demand planning.

```sql
SELECT 
    p.ProductID,
    p.SKU,
    p.ProductName,
    SUM(oi.Quantity) AS UnitsSold,
    SUM(oi.Quantity * oi.UnitPrice) AS TotalLineRevenue,
    p.StockQuantity AS CurrentRemainingStock
FROM Order_Items oi
JOIN Products p ON oi.ProductID = p.ProductID
JOIN Orders o ON oi.OrderID = o.OrderID
WHERE o.OrderStatus != 'Cancelled'
GROUP BY p.ProductID, p.SKU, p.ProductName, p.StockQuantity
ORDER BY UnitsSold DESC
LIMIT 10;
