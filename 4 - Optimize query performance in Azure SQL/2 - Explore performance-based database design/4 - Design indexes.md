## 🔑 **What is an Index?**

An **index** is like a **lookup table** that helps SQL Server find rows quickly instead of scanning the entire table.

- Without an index → **Full Table Scan** → Slower
- With an index → **Seek to Exact Row(s)** → Faster

---

## 📂 **Types of Indexes**

### 1. ✅ **Clustered Index**

- **Defines the logical order of the rows in the table.**
- The **data IS the index** → only **one clustered index per table.**
- Table with no clustered index = **heap.**

▶️ Example:

```sql
CREATE CLUSTERED INDEX IX_Customers_ID ON Customers(CustomerID);
```

Now the data is stored sorted by `CustomerID`.

- **Best For:** Primary key, frequently used in ORDER BY queries, or columns that are used in range queries.
- **Tip:** Keep the clustered index key **narrow** (few bytes) and **unique**.

---

### 2. ✅ **Nonclustered Index**

- **Separate structure** from the table data.
- Contains **index key(s)** and **a pointer** to the actual table rows.
- You can create **multiple nonclustered indexes** on a table.

▶️ Example:

```sql
CREATE NONCLUSTERED INDEX IX_Customers_Email ON Customers(Email);
```

- **Key Lookup Problem:** If your query needs columns **not in the index**, SQL Server will still access the table (called a **Key Lookup**).

▶️ Covering Index Example:

```sql
CREATE NONCLUSTERED INDEX IX_Customers_Email_Phone
ON Customers(Email)
INCLUDE (PhoneNumber, FullName);
```

This lets SQL Server get **all needed columns** from the index, avoiding a lookup.

---

### 3. ✅ **Filtered Index**

Index on **a subset of rows**, based on a WHERE condition.
Great for columns where most rows share the same value.

▶️ Example:

```sql
CREATE NONCLUSTERED INDEX IX_Employees_Current
ON HumanResources.Employee(BusinessEntityID)
WHERE CurrentFlag = 1;
```

This speeds up queries **only for currently employed staff.**

---

### 4. ✅ **Columnstore Index**

Best for **analytics (OLAP, Data Warehousing)**.

- Stores data **column-wise**, compresses similar data efficiently.
- Reads only needed columns → less I/O.

▶️ Example:

```sql
CREATE CLUSTERED COLUMNSTORE INDEX CCI_Sales ON FactSales;
```

⚙️ Benefits:

- Massive speed for aggregations (`SUM`, `AVG`, etc.).
- Processes data in **batches (\~900 rows at a time).**

---

## 🌳 **Index Internal Structure: B-tree**

SQL Server **B-tree indexes** are organized as:

- **Root Node:** Starting point
- **Intermediate Nodes:** Help navigate
- **Leaf Nodes:** Actual data (or pointers in nonclustered)

---

## ⚠️ **Index Design Tips**

| Principle                               | Explanation                                             |
| --------------------------------------- | ------------------------------------------------------- |
| Keep clustered keys narrow              | Reduces size of related indexes and storage.            |
| Index for your most common queries      | Don't create indexes you won't use.                     |
| Use integers where possible             | Faster comparisons and smaller index sizes.             |
| Index columns used in JOIN/WHERE        | Especially useful for foreign keys and filters.         |
| Avoid indexing highly updated columns   | Index maintenance overhead grows with frequent updates. |
| Minimize number of columns in index key | Reduces size, improves performance.                     |

---

## ⚙️ Example Scenario: Solving a Key Lookup

### Problem:

```sql
SELECT ProductName, Price FROM Products WHERE ProductID = 5;
```

- Nonclustered index on `ProductID`, but no index includes `Price`.
- SQL Server first **uses the index on ProductID**, then performs a **Key Lookup** to get the Price.

### Fix:

```sql
CREATE NONCLUSTERED INDEX IX_Products_ProductID
ON Products(ProductID)
INCLUDE (ProductName, Price);
```

---

## 🛢️ **Columnstore Example for Warehouses**

For analytic queries like:

```sql
SELECT SUM(SalesAmount) FROM FactResellerSales;
```

- Columnstore will **scan only the SalesAmount column**, not the whole row → much faster.

- Example load time difference for **1 million rows**:

  - Columnstore: 8 seconds
  - Rowstore (clustered index): 20 seconds

---

## 🔁 **Delta Store in Columnstore**

If you load **< 102,400 rows**, they temporarily go to a **row-based delta store**.
SQL Server later **compresses & moves** them into the columnstore (**Tuple Mover process**).

---

## 🏁 **Summary**

| Index Type         | Best For                                 | Example Use                |
| ------------------ | ---------------------------------------- | -------------------------- |
| Clustered Index    | Primary sort & frequently queried fields | CustomerID                 |
| Nonclustered Index | Additional queries, lookup columns       | Email, LastName            |
| Filtered Index     | Partial queries on common column values  | Active employees           |
| Columnstore Index  | Analytics, large aggregations            | Fact tables in a warehouse |

---
