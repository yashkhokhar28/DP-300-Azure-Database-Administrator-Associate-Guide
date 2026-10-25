## 🔍 **What is Performance-Based Database Design?**

**Database design directly affects performance**. Even if you're not the developer of the system (for example, if you're a DBA managing a third-party app), **understanding and tuning the design** is key to improving query speed, reducing resource usage, and supporting the expected workload.

---

## 🎯 **Key Design Approaches Based on Workload Type**

| Workload Type                            | Design Focus                                                          | Example                                                                                                                 |
| ---------------------------------------- | --------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **OLTP (Online Transaction Processing)** | Fast small transactions, frequent reads/writes, strong data integrity | - Normalize tables (reduce data duplication)<br>- Index frequently queried columns<br>- Keep transactions short         |
| **Data Warehouse (OLAP)**                | Complex analytical queries, large reads, less frequent writes         | - Denormalize for fewer joins<br>- Use wide tables with aggregated data<br>- Partition large tables for faster scanning |

### Example:

- **OLTP design:**

  - `Orders`, `Customers`, and `Products` are in separate tables → fast inserts/updates.
  - Example query:

    ```sql
    SELECT FirstName, LastName FROM Customers WHERE CustomerID = 100;
    ```

- **Data Warehouse design:**

  - Combine into a **fact table** and **dimension tables**, sometimes denormalized.
  - Example query:

    ```sql
    SELECT SUM(SalesAmount) FROM SalesFact WHERE Year = 2024 GROUP BY Region;
    ```

---

## 🔑 **Critical Design Choices for Performance**

### 1. **Choose the Right Datatypes**

- Smaller datatypes = less storage = faster queries.
- Example:

  - ✅ `INT` for numbers (4 bytes)
  - ❌ Avoid `VARCHAR(10)` for numbers—it wastes space and slows sorting.

Example:

```sql
CREATE TABLE Employees (
    EmployeeID INT,        -- Efficient
    PhoneNumber VARCHAR(10) -- For string data like phone numbers
);
```

---

### 2. **Indexing**

Indexes make searches faster but slow down writes. Create indexes based on query patterns.

Example:

```sql
CREATE INDEX idx_lastname ON Employees(LastName);
```

Without an index, this query will scan the whole table:

```sql
SELECT * FROM Employees WHERE LastName = 'Smith';
```

With an index, SQL Server can quickly find the rows for 'Smith'.

---

### 3. **Partitioning for Large Tables**

- Split large tables into smaller, manageable pieces (partitions).
- Example: Partition `Sales` by year, so queries for one year scan only that year's partition.

---

### 4. **Denormalization in Data Warehouses**

In OLAP, reducing joins improves performance.
Example denormalized table:

| OrderID | CustomerName | ProductName | TotalAmount |
| ------- | ------------ | ----------- | ----------- |
| 1       | John Smith   | Laptop      | 1500        |

Instead of joining `Orders`, `Customers`, and `Products`.

---

## ✔️ **Summary of Best Practices**

- ✅ Design for the workload (**transactional or analytical**).
- ✅ Pick correct **datatypes** for efficiency.
- ✅ Add **indexes** on columns that are frequently searched.
- ✅ Partition or denormalize **when needed for performance**.

---
