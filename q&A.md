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


## Step 1: Base Measure

```DAX id="rwc_base"
Total Sales =
SUM(factSales[SalesAmount])
```



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



## SQL Equivalent (Good for Interviews)

```sql id="rwc_sql"
RANK() OVER (
    PARTITION BY Region, Category
    ORDER BY Sales DESC
)
```



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



## Top N within Region & Category (Common Follow-up)

```DAX id="rwc_topn"
Top3 Within Region Category =
IF(
    [Rank Within Region Category] <= 3,
    [Total Sales]
)
```



## Interview Tip

**When to use ALLEXCEPT?**

| Scenario              | Function    |
| --------------------- | ----------- |
| Rank within group     | ALLEXCEPT   |
| Rank overall          | ALL         |
| Rank based on slicers | ALLSELECTED |

---

6. This is a **very common interview comparison**. These three functions look similar but are used for **different purposes**.

Let’s understand with **definition, syntax, when to use, and examples**.



# ISINSCOPE vs HASONEVALUE vs ISFILTERED

## 1. ISINSCOPE()

### Syntax

```DAX
ISINSCOPE(<column>)
```

### Definition

Checks whether a column is present at the **current level of a hierarchy in a visual**.

### When to use

* Matrix or hierarchy visuals
* Drill-down scenarios
* Level-based calculations



### Example

Hierarchy: Category → Subcategory

```DAX
Sales Display =
IF(
    ISINSCOPE(Dim_Product[Subcategory]),
    [Total Sales],
    BLANK()
)
```

Shows sales only at Subcategory level.



## 2. HASONEVALUE()

### Syntax

```DAX
HASONEVALUE(<column>)
```

### Definition

Returns TRUE if the column has **only one value in the current filter context**.

### When to use

* When exactly one value is selected
* Validation for single selection



### Example

```DAX
Sales (Single Category) =
IF(
    HASONEVALUE(Dim_Product[Category]),
    [Total Sales]
)
```



## 3. ISFILTERED()

### Syntax

```DAX
ISFILTERED(<column_or_table>)
```

### Definition

Returns TRUE if a column or table is **directly filtered** by:

* Slicer
* Visual
* Filter pane



### Example

```DAX
Sales When Filtered =
IF(
    ISFILTERED(Dim_Product[Category]),
    [Total Sales],
    BLANK()
)
```


# Key Differences

| Function    | Checks                     |
| ----------- | -------------------------- |
| ISINSCOPE   | Hierarchy level in visual  |
| HASONEVALUE | Exactly one value selected |
| ISFILTERED  | Any direct filter applied  |



# Example Scenario

If slicer selects:

* Category = Electronics

| Function              | Result                                            |
| --------------------- | ------------------------------------------------- |
| HASONEVALUE(Category) | TRUE                                              |
| ISFILTERED(Category)  | TRUE                                              |
| ISINSCOPE(Category)   | TRUE only if Category is used in visual hierarchy |



If multiple categories selected:

| Function    | Result                  |
| ----------- | ----------------------- |
| HASONEVALUE | FALSE                   |
| ISFILTERED  | TRUE                    |
| ISINSCOPE   | Depends on visual level |



# When to Use (Interview Tip)

| Requirement            | Function    |
| ---------------------- | ----------- |
| Check hierarchy level  | ISINSCOPE   |
| Check single selection | HASONEVALUE |
| Check if filter exists | ISFILTERED  |



# Interview Answer (2-Min Version)

ISINSCOPE is used to detect whether a column is currently at a specific hierarchy level in a visual, mainly for drill-down or matrix scenarios. HASONEVALUE checks whether only one value exists in the current filter context and is used when a calculation requires a single selection. ISFILTERED checks whether a column or table is directly filtered by a slicer, visual, or filter pane. Each function serves a different purpose: hierarchy detection, single-value validation, and filter detection respectively.



# Important Interview Follow-up

**Which is best for Matrix level control?**
Answer: **ISINSCOPE**

----

7. ## How do you create Dynamic Measure Selection in Power BI?

### Concept

Dynamic measure selection allows users to:

> Choose a metric (Sales, Profit, Quantity, etc.) from a slicer and display the selected measure dynamically.

This is implemented using:

* **Disconnected Table**
* **SELECTEDVALUE()**
* **SWITCH()**



## Step-by-Step Implementation

### Step 1: Create a Disconnected Table

Create a table for measure names:

```DAX id="1m8w8f"
Measure Selector =
DATATABLE(
    "Metric", STRING,
    {
        {"Sales"},
        {"Profit"},
        {"Quantity"}
    }
)
```

**Important:**
Do **not create any relationship** with other tables.



### Step 2: Create Base Measures

```DAX id="c0n3f8"
Total Sales = SUM(Fact_Sales[SalesAmount])

Total Profit = SUM(Fact_Sales[Profit])

Total Quantity = SUM(Fact_Sales[Quantity])
```



### Step 3: Capture User Selection

#### SELECTEDVALUE Syntax

```DAX id="q9z6y2"
SELECTEDVALUE(<column>, [alternate_result])
```

Returns the selected value from slicer.


### Step 4: Create Dynamic Measure using SWITCH

```DAX id="k3n1x4"
Selected Metric Value =
SWITCH(
    SELECTEDVALUE('Measure Selector'[Metric]),
    "Sales", [Total Sales],
    "Profit", [Total Profit],
    "Quantity", [Total Quantity],
    BLANK()
)
```



### Step 5: Use in Report

1. Add **Measure Selector[Metric]** to a slicer
2. Add **Selected Metric Value** to visuals

Now visuals change based on user selection.


## How It Works Internally

1. User selects "Sales" in slicer
2. Filter applies to disconnected table
3. `SELECTEDVALUE()` returns "Sales"
4. `SWITCH()` returns `[Total Sales]`



## Optional: Dynamic Title

```DAX id="v4k2m7"
Selected Metric Title =
"Showing: " &
SELECTEDVALUE('Measure Selector'[Metric], "Sales")
```



## Functions Used

| Function      | Purpose                      |
| ------------- | ---------------------------- |
| DATATABLE     | Create disconnected table    |
| SELECTEDVALUE | Get slicer selection         |
| SWITCH        | Return corresponding measure |



## When to Use Dynamic Measure Selection?

* KPI switching (Sales/Profit)
* Financial reports
* Dashboard simplification
* Avoid multiple visuals



## Interview Answer (2-Min Version)

Dynamic measure selection is implemented using a disconnected table that contains measure names. The selected value from the slicer is captured using SELECTEDVALUE, and a SWITCH function is used to return the corresponding measure dynamically. This approach allows a single visual to display different metrics based on user selection and is commonly used for KPI switching and interactive dashboards.



## Interview Tip

If asked:
**Why use a disconnected table?**

Answer:
Because it is used only to capture user input and control logic through DAX, not to filter data directly.

----

8. 






