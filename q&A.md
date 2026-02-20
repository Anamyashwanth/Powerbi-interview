1. How do you design a data model for a large dataset?
  <br>Here is a **2-minute interview answer** you can deliver confidently:
<br>**Answer:**

When designing a data model for a large dataset, my approach focuses on **performance, scalability, and usability for reporting**.

First, I start by understanding the **business requirements** — what metrics are needed, the level of granularity (daily, transaction-level), and how the data will be used in tools like Power BI.

Next, for large analytical datasets, I use a **Dimensional Model (Star Schema)**.
I separate the data into:

* **Fact tables** – large transactional data like sales or orders
* **Dimension tables** – descriptive data like product, customer, and date

This structure improves query performance and makes the model easy for users to understand.

To handle large data volumes, I implement:

* **Incremental loading** instead of full loads, using a watermark or last modified date
* **Partitioning**, usually by date, to improve query speed and maintenance
* **Indexes and surrogate keys** to optimize joins and filtering

If the data volume is very high, I also create **aggregate tables** (daily or monthly summaries) to reduce query time.

From an architecture perspective, I follow a **Bronze–Silver–Gold approach**:

* Bronze: Raw data
* Silver: Cleaned and transformed data
* Gold: Final star schema for reporting

Finally, I optimize the model by:

* Removing unnecessary columns
* Using integer keys instead of text
* Avoiding complex relationships like many-to-many

This approach ensures the data model is **scalable, high-performance, and optimized for BI tools like Power BI or Fabric**.

A surrogate key is a system-generated unique identifier used as the primary key in a table instead of a business/natural key.

It has no business meaning and is usually:

Integer (IDENTITY, SEQUENCE)

Auto-generated

Used for relationships between tables

---

2. Difference between Star Schema and Snowflake Schema?
  <br>Star schema is a dimensional model where a central fact table is connected to denormalized dimension tables, while in a snowflake schema, dimension tables are normalized into multiple related tables.

Star schema has fewer tables, simpler relationships, and faster query performance, whereas snowflake schema reduces data redundancy but increases complexity and requires more joins.

In Power BI, star schema is recommended because it improves performance, simplifies DAX calculations, and provides better compression and usability.

---
3. Difference between Calculated Column and Measure?
<br>  A calculated column is computed during data refresh and stored in the model, using row context. It is used when we need a column for filtering, grouping, or relationships. However, it increases model size.

A measure is calculated at query time based on filter context and is not stored in memory. Measures are used for aggregations in visuals and provide better performance. In most scenarios, measures are preferred over calculated columns to optimize the model.

------
4. How do you carry filters from one report to another?

<br>If navigation is within the same report, I use Drillthrough to pass filter context. For different reports, I use cross-report drillthrough or URL filtering. Drillthrough passes the user’s filter and slicer context automatically. DAX functions like ALLSELECTED only control filter behavior within a report and are not used for transferring filters.

| Scenario                | Solution                  |
| ----------------------- | ------------------------- |
| Page → Page (same PBIX) | Drillthrough              |
| Report → Report         | Cross-report Drillthrough |
| Dynamic navigation      | URL Filters               |

----
5. ## Rank within Region & Category (Multi-Group Ranking)

**Requirement:**
Rank **Products** within each **Region and Category** based on Sales.

Ranking should restart for every **Region–Category combination**.

Example:

| Region | Category    | Product | Sales | Rank |
| ------ | ----------- | ------- | ----- | ---- |
| East   | Electronics | A       | 500   | 1    |
| East   | Electronics | B       | 400   | 2    |
| East   | Furniture   | C       | 300   | 1    |
| West   | Electronics | A       | 450   | 1    |

---

## Step 1: Base Measure

```DAX id="rwc_base"
Total Sales =
SUM(factSales[SalesAmount])
```

---

## Step 2: Rank within Region & Category

```DAX id="rwc_main"
Rank Within Region Category =
RANKX(
    ALLEXCEPT(
        factSales,
        factSales[Region],
        factSales[Category]
    ),
    [Total Sales],
    ,
    DESC,
    DENSE
)
```

---

## How it Works (Interview Explanation)

### ALLEXCEPT

```DAX id="rwc_logic"
ALLEXCEPT(factSales, Region, Category)
```

* Removes all filters
* Keeps only:

  * Region
  * Category
* So ranking happens **within each Region–Category group**

### RANKX

* Ranks products based on Total Sales
* `DESC` → Highest sales = Rank 1
* `DENSE` → No rank gaps

---

## SQL Equivalent (Good for Interviews)

```sql id="rwc_sql"
RANK() OVER (
    PARTITION BY Region, Category
    ORDER BY Sales DESC
)
```

---

## Best Practice (Star Schema)

If you have dimension tables:

```DAX id="rwc_best"
Rank Within Region Category =
RANKX(
    ALLEXCEPT(
        Dim_Product,
        Dim_Region[Region],
        Dim_Category[Category]
    ),
    [Total Sales],
    ,
    DESC,
    DENSE
)
```

---

## Top N within Region & Category (Common Follow-up)

```DAX id="rwc_topn"
Top3 Within Region Category =
IF(
    [Rank Within Region Category] <= 3,
    [Total Sales]
)
```

---

## Interview Tip

**When to use ALLEXCEPT?**

| Scenario              | Function    |
| --------------------- | ----------- |
| Rank within group     | ALLEXCEPT   |
| Rank overall          | ALL         |
| Rank based on slicers | ALLSELECTED |

---

6.
7.
8. 






