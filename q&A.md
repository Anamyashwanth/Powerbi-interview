1. How do you design a data model for a large dataset?
  Here is a **2-minute interview answer** you can deliver confidently:
**Answer:**

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

-------

2. Difference between Star Schema and Snowflake Schema?
  Star schema is a dimensional model where a central fact table is connected to denormalized dimension tables, while in a snowflake schema, dimension tables are normalized into multiple related tables.

Star schema has fewer tables, simpler relationships, and faster query performance, whereas snowflake schema reduces data redundancy but increases complexity and requires more joins.

In Power BI, star schema is recommended because it improves performance, simplifies DAX calculations, and provides better compression and usability.

3. 
