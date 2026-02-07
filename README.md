# 🚀 Databricks Metric Views, Genie & Research Agent - Complete Guide

A production-ready implementation showing how to build AI-powered analytics with **Databricks Metric Views**, **Genie**, and **Research Agent**.

## 📁 Repository Structure

```
├── data/                                    # Sample datasets
│   ├── retail_transactions.csv             # 100 transaction records
│   └── store_performance.csv               # 100 store performance records
├── src/metric_views/                        # Metric View YAML definitions
│   ├── retail_transactions_metric_view.yaml
│   └── retail_store_performance_metric_view.yaml
├── docs/                                    # Documentation
│   ├── linkedin_article_metric_views_genie.md
│   └── images/                              # Architecture diagrams
└── README.md                                # This file
```

---

## 🚀 Quick Start Guide

### Prerequisites

- Databricks workspace with Unity Catalog enabled
- A catalog and schema where you have CREATE TABLE permissions
- Access to Databricks SQL or a cluster

---

## Step 1: Data Ingestion

### 1.1 Upload CSV Files to Unity Catalog Volume

```python
# In a Databricks notebook

# Create a volume (if not exists)
spark.sql("""
  CREATE VOLUME IF NOT EXISTS my_catalog.my_schema.raw_data
""")

# Upload files via UI: Catalog → Your Schema → Volumes → Upload Files
# Or use dbutils:
# dbutils.fs.cp("file:/local/path/retail_transactions.csv", "/Volumes/my_catalog/my_schema/raw_data/")
```

### 1.2 Create Delta Tables

```python
# Transaction Data
df_transactions = (spark.read
    .option("header", "true")
    .option("inferSchema", "true")
    .csv("/Volumes/my_catalog/my_schema/raw_data/retail_transactions.csv")
)

df_transactions.write.mode("overwrite").saveAsTable("my_catalog.my_schema.retail_business_table_1")

# Store Performance Data
df_performance = (spark.read
    .option("header", "true")
    .option("inferSchema", "true")
    .csv("/Volumes/my_catalog/my_schema/raw_data/store_performance.csv")
)

df_performance.write.mode("overwrite").saveAsTable("my_catalog.my_schema.retail_business_table_2")
```

### 1.3 Add Table Comments

```sql
COMMENT ON TABLE my_catalog.my_schema.retail_business_table_1 IS 
  'Transaction-level retail data including customer purchases, products, and payment methods';

COMMENT ON TABLE my_catalog.my_schema.retail_business_table_2 IS 
  'Monthly aggregated store performance metrics including revenue, costs, and profitability';
```

---

## Step 2: Create Metric Views

### 2.1 Via Catalog Explorer UI

1. Navigate to **Catalog** → Your Schema
2. Click **Create** → **Metric View**
3. Copy the YAML from `src/metric_views/retail_transactions_metric_view.yaml`
4. **Important:** Update the `source` line to match your catalog/schema:
   ```yaml
   source: my_catalog.my_schema.retail_business_table_1
   ```
5. Click **Create**

Repeat for `retail_store_performance_metric_view.yaml`.

### 2.2 Via SQL

```sql
CREATE METRIC VIEW my_catalog.my_schema.retail_transactions_metrics
AS '
version: 1.1
source: my_catalog.my_schema.retail_business_table_1

dimensions:
  - name: store_city
    expr: StoreCity
    comment: City where the store is located
  - name: customer_segment
    expr: CustomerSegment
    comment: Customer loyalty segment

measures:
  - name: total_revenue
    expr: SUM(TotalAmount)
    comment: Total revenue from all transactions
  - name: transaction_count
    expr: COUNT(*)
    comment: Number of transactions
';
```

### 2.3 Grant Permissions

```sql
GRANT SELECT ON METRIC VIEW my_catalog.my_schema.retail_transactions_metrics TO `analysts`;
GRANT SELECT ON METRIC VIEW my_catalog.my_schema.retail_store_performance_metrics TO `analysts`;
```

---

## Step 3: Create Genie Space

1. Navigate to **Workspace** → **New** → **Genie Space**
2. Name: "Retail Analytics Assistant"
3. Add data sources:
   - Select your metric views from Unity Catalog
4. Add instructions (optional):
   ```
   You are a retail analytics assistant.
   - Customer segments: VIP, Premium, Regular, New
   - Profit margin target: 50% or higher is healthy
   - Cost-to-revenue ratio target: Below 0.45 is efficient
   ```
5. Click **Create**

### Test Queries

Try these questions in your Genie Space:

| Question | Expected Result |
|----------|-----------------|
| "What was total revenue last month?" | Aggregated revenue |
| "Which stores have profit margins below 50%?" | Filtered store list |
| "Compare VIP vs Regular customer spending" | Segmented comparison |
| "Give me a 360 view of my business" | Comprehensive analysis |

---

## Step 4: Set Up Alerts

### 4.1 Create Alert Query

In **SQL Editor**, create a new query:

```sql
SELECT 
  store_id,
  city,
  average_profit_margin,
  cost_to_revenue_ratio
FROM my_catalog.my_schema.retail_store_performance_metrics
WHERE average_profit_margin < 45
   OR cost_to_revenue_ratio > 0.55
ORDER BY average_profit_margin ASC
```

### 4.2 Configure Alert

1. Save the query
2. Click the **Alert** button (bell icon)
3. Configure:
   - **Trigger:** Query returns results
   - **Frequency:** Every 1 hour
   - **Destination:** Slack, Email, or Webhook

---

## Step 5: Use Research Agent

Research Agent is Genie's advanced analytical capability for answering "why" questions.

### Trigger Research Agent

Ask questions like:
- "Why did Store STR008 profit margin drop?"
- "Analyze the revenue decline in Q3"
- "Explain the difference between top and bottom performing stores"

### What Research Agent Does

1. **Breaks down** complex questions into sub-queries
2. **Investigates** multiple dimensions and time periods
3. **Correlates** findings across metrics
4. **Synthesizes** insights with actionable recommendations

> 📚 [Research Agent Documentation](https://docs.databricks.com/aws/en/genie/research-agent)

---

## 📊 Data Schema

### retail_transactions.csv

| Column | Type | Description |
|--------|------|-------------|
| TransactionID | String | Unique transaction ID |
| StoreID | String | Store identifier (STR001-STR020) |
| StoreCity | String | City location |
| CustomerID | String | Customer identifier |
| CustomerSegment | String | VIP, Premium, Regular, New |
| TransactionDateTime | Timestamp | Transaction date/time |
| ProductID | String | Product code |
| ProductName | String | Product description |
| Category | String | Product category |
| Quantity | Integer | Items purchased |
| UnitPrice | Decimal | Price per unit |
| DiscountPercent | Decimal | Discount applied |
| TotalAmount | Decimal | Final amount |
| PaymentMethod | String | Payment type |

### store_performance.csv

| Column | Type | Description |
|--------|------|-------------|
| StoreID | String | Store identifier |
| City | String | Store location |
| Month | String | YYYY-MM format |
| MonthlyRevenue | Decimal | Total monthly revenue |
| MonthlyTransactions | Integer | Transaction count |
| AvgBasketSize | Decimal | Average basket value |
| UniqueCustomers | Integer | Unique customer count |
| EmployeeCount | Integer | Staff count |
| OperatingCosts | Decimal | Monthly costs |
| ProfitMarginPercent | Decimal | Profit margin % |

---

## 📚 Resources

### Official Documentation

- [Metric Views Overview](https://docs.databricks.com/en/metric-views/)
- [Create Metric Views](https://docs.databricks.com/en/metric-views/create/)
- [YAML Syntax Reference](https://docs.databricks.com/en/metric-views/yaml-ref)
- [Databricks Genie](https://docs.databricks.com/en/genie/)
- [Research Agent](https://docs.databricks.com/aws/en/genie/research-agent)
- [SQL Alerts](https://docs.databricks.com/en/sql/user/alerts/)

---

## 🤝 Contributing

Feel free to open issues or submit PRs to improve the metric view definitions or add new use cases.

---

## 📝 License

MIT License

---

## 👤 Author

**Mehdi Wissad**  
Databricks Solution Architect

---

*Built with ❤️ using Databricks Metric Views, Genie & Research Agent*
