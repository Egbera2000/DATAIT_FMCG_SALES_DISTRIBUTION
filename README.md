# DATAIT_FMCG_SALES_DISTRIBUTION

The DATAIT FMCG_SALES_DISTRIBUTION repository, houses my SQL queries for my final project during the Cohort 5.0 period where i solved 10 problems in the FMCG industry faces with respect to their sales department and also ensured that sales rose from 15% to about 75% profit Margin

The SQL Server capstone project is built on a simulated FMCG (Fast-Moving Consumer Goods) distribution dataset, covering order management, agent productivity, payment reconciliation, retailer intelligence, and product/category sales analysis.

This project was completed as part of the DataIT SQL Server course and demonstrates practical use of CRUD operations, joins, aggregation, GROUP BY/HAVING, stored procedures, and views on a multi-table relational schema.

# Table of Contents

* Dataset Overview
* Tools
* Project Structure
* PROJECT 1: Daily Run-Rate Monitor
* PROJECT 2: Agent Productivity Review
* PROJECT 3: Order-to-Cash Audit
* PROJECT 4: Retailer Intelligence
* PROJECT 5: What's Selling and Why
* Skills Demonstrated
* How to Run

# Dataset Overview

The database, DataIT_FMCG, models a distribution business with five core tables:

# Table	Description

| Table | Description |
|---|---|
| `Agents` | Sales agents — code, name, phone, home state/city, role, hire date, status |
| `Retailers` | Retail customers — business name, owner, phone, channel, payment preference, credit limit, location |
| `SalesOrders` | Order headers — order number, retailer, agent, order date, fulfillment type, stock point, status, cancel reason, net amount |
| `SalesOrderLines` | Order line items — SKU code/name, category, unit of measure, quantity, unit price, line total |
| `Payments` | Payment records — linked order, payment date, amount, method, reference, status |

# Tools

- Microsoft SQL Server 
- SQL Server Management Studio (SSMS)

# Project Structure

The work is organized into five mission projects, each targeting a different business question. Every mission follows the same 10-question pattern: Create → Update → Delete → Sort → Aggregate → Group By → Join(s) → Stored Procedure / View.

# PROJECT 1: Daily Run-Rate Monitor

Order pipeline health — volumes, statuses, and fulfillment performance.

- Inserted, updated, and cleaned up order header records
- Pulled the last 30 days of orders with retailer and agent names attached
- Broke down order counts by status and by day
- Calculated delivered GMV (gross merchandise value) by stock point
- Built per-agent delivery metrics (count + average order value)
- Reported on cancelled orders with reason, retailer, and agent
- Wrote usp_CreateOrderHeaderB, a stored procedure to insert new order headers

Sample query — daily order counts, last 30 days:

```sql
 SELECT CAST(OrderDate AS DATE) AS OrderDay, COUNT(*) AS Orders
    FROM dbo.SalesOrders
    WHERE OrderDate >= DATEADD(DAY, -30, GETDATE())
    GROUP BY CAST(OrderDate AS DATE)
 ORDER BY OrderDay;
```

# PROJECT 2: Agent Productivity Review

Agent roster management and performance against delivered sales.

- Managed the agent roster (create, suspend, delete exited agents with no order history)
- Listed the most recently hired agents
- Counted agents by role, and flagged states with more than 5 active agents (HAVING)
- Computed delivered GMV per agent (INNER JOIN)
- Identified agents with zero delivered orders (LEFT JOIN)
- Listed every order alongside its agent, including orphaned records (RIGHT JOIN)
- Wrote Usp_SetAgentstatusB, a stored procedure to update an agent's status by code

Sample query — delivered GMV per agent:

```sql
 SELECT a.AgentCode, a.FullName, SUM(o.NetAmount) AS DeliveredGMV
     FROM dbo.Agents a
     JOIN dbo.SalesOrders o ON o.AgentID = a.AgentID
     WHERE o.Status = 'Delivered'
     GROUP BY a.AgentID, a.AgentCode, a.FullName
 ORDER BY DeliveredGMV DESC;
```

# PROJECT 3: Order-to-Cash Audit

Tracing revenue from order to confirmed payment.

- Recorded and reconciled payment transactions
- Purged old reversed payments (>180 days)
- Listed the latest 50 payments with order number and retailer
- Counted payments by method and totaled daily payments over 30 days
- Summed confirmed payments per retailer against their credit limit
- Calculated outstanding balance per order (NetAmount − total confirmed paid)
- Flagged delivered orders with no confirmed payment — a key collections risk report

Sample query — outstanding balance per order:

```sql
SELECT o.OrderNumber, o.NetAmount, ISNULL(p.TotalPaid, 0) AS TotalPaid,
       o.NetAmount - ISNULL(p.TotalPaid, 0) AS Outstanding
FROM dbo.SalesOrders o
LEFT JOIN (
    SELECT OrderID, SUM(Amount) AS TotalPaid
    FROM dbo.Payments
    WHERE Status = 'Confirmed'
    GROUP BY OrderID
)p ON p.OrderID = o.OrderID
ORDER BY Outstanding DESC;
```
PROJECT 4: Retailer Intelligence

Retailer segmentation, credit exposure, and ordering behavior.

- Onboarded new retailers and adjusted credit limits by channel (bulk 15% increase for Supermarkets)
- Removed stale retailers (>2 years old, no orders)
- Ranked retailers by credit limit
- Counted retailers by channel, and by state with average credit limit
- Counted orders per retailer (INNER JOIN)
- Identified retailers with delivered orders vs. none (LEFT JOIN)
- Found HoReCa retailers in Abuja (FCT) with their most recent order date
- Built VW_RetailerOrderStats, a view summarizing total orders, delivered orders, and delivered GMV per retailer

View — VW_RetailerOrderStats:

```sql
CREATE OR ALTER VIEW dbo.VW_RetailerOrderStats AS
SELECT r.RetailerID, r.BusinessName,
       COUNT(o.OrderID) AS TotalOrders,
       SUM(CASE WHEN o.Status = 'Delivered' THEN 1 ELSE 0 END) AS DeliveredOrders,
       SUM(CASE WHEN o.Status = 'Delivered' THEN o.NetAmount ELSE 0 END) AS DeliveredGMV
FROM dbo.Retailers r
LEFT JOIN dbo.SalesOrders o ON o.RetailerID = r.RetailerID
GROUP BY r.RetailerID, r.BusinessName;
```

PROJECT 5: What's Selling and Why

Product-level performance across the order-line grain.

- Added, adjusted, and cleaned up order line items
- Ranked the most expensive line items by line total
- Summed quantity sold by unit of measure (UOM)
- Ranked the top 10 SKUs by revenue for delivered orders
- Split category revenue by state (SKU → order → retailer)
- Flagged orders with no line items (data-quality check)
- Filtered for a specific SKU/UOM/quantity combination
- Built VW_OrderBasket, a view exposing the full order-line basket detail

View — VW_OrderBasket:

```sql
CREATE OR ALTER VIEW dbo.VW_OrderBasket AS
SELECT
     o.OrderNumber, l.SKUName, l.Category, l.UOM, l.Qty, l.UnitPrice, l.LineTotal
     FROM dbo.SalesOrderLines l
JOIN dbo.SalesOrders o ON o.OrderID = l.OrderID;
```

# Skills Demonstrated

- CRUD operations wrapped in transactions (BEGIN TRAN / ROLLBACK) for safe testing
- Filtering, sorting, and TOP N queries
- Aggregate functions: COUNT, SUM, AVG
- GROUP BY and HAVING for segmented reporting
- INNER JOIN, LEFT JOIN, RIGHT JOIN across a 5-table schema
- Correlated subqueries with NOT EXISTS
- Conditional aggregation (CASE WHEN inside SUM)
- Stored procedures with parameters for repeatable inserts/updates
- Views for reusable reporting logic
  
# How to Run

- Restore or create the DataIT_FMCG database in SQL Server.
- Open the .sql script in SSMS or Azure Data Studio.
- Run USE DataIT_FMCG; first.
- Execute each mission section in order — write operations are wrapped in BEGIN TRAN ... ROLLBACK so they can be run safely without permanently altering the data. Remove or change ROLLBACK to COMMIT to persist changes.
