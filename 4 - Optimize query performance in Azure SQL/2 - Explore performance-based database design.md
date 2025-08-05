## 🚀 **What Is Performance-Based Database Design?**

It’s not just about making your database “work” — it’s about making it **fly**.

You wouldn’t build a race car using city bus blueprints, right? Same with databases. Your design must fit the **type of workload** — or you’ll crash into performance issues later.

🔧 Even if you're not coding the app, as a DBA or architect, **tuning the schema** can make or break performance.

---

## 🛒 **OLTP vs. OLAP — Design Like You Mean It**

| Workload Type            | Core Design Vibe                                        | Real-World Analogy                                          | Design Musts                                     |
| ------------------------ | ------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------ |
| **OLTP** (Transactional) | Think fast! High-speed reads/writes, integrity is king. | **E-commerce checkout system** (Amazon cart)                | Normalize, index key columns, short transactions |
| **OLAP** (Analytical)    | Crunch heavy data for insights. Less write, more read.  | **Sales dashboard/reporting** (monthly Amazon sales trends) | Denormalize, wide tables, partitioning           |

🧠 Example:

- **OLTP**:
  Customers, Orders, Products — all in separate, normalized tables.

  ```sql
  SELECT FirstName FROM Customers WHERE CustomerID = 101;
  ```

- **OLAP**:
  SalesFact table contains pre-aggregated info.

  ```sql
  SELECT SUM(SalesAmount) FROM SalesFact WHERE Region = 'West' GROUP BY Year;
  ```

---

## 🧩 **Design Choices That Actually Matter**

### 1. **Data Types: Don't Be Lazy**

💡 Right size = right speed.

Bad:

```sql
OrderAmount VARCHAR(50) -- slow, bloated
```

Good:

```sql
OrderAmount DECIMAL(10,2) -- fast, precise
```

Real-World: Storing a phone number as INT is like storing a license plate as a float. Just... why?

---

### 2. **Indexing: Your Query's Best Friend (or Worst Enemy)**

- Indexes = Fast reads ✅
- But slow writes if overused ❌

```sql
-- Fast lookup
CREATE INDEX idx_email ON Users(Email);
```

👎 Bad:

```sql
WHERE CAST(UserID AS VARCHAR) = '10' -- breaks index seek
```

👍 Good:

```sql
WHERE UserID = 10 -- index seek enabled
```

---

### 3. **Partitioning: Slice It Right**

Split big tables like `Sales` by `Year`, so annual reports don’t choke on 10 years of data.

```sql
-- Each partition is like a drawer in a filing cabinet
SELECT * FROM Sales WHERE Year = 2024;
```

---

### 4. **Denormalization: When Speed > Purity**

For OLAP, fewer joins = faster queries.

👎 Normalized:

- Separate `Orders`, `Products`, `Customers`

👍 Denormalized:

```plaintext
OrderID | CustomerName | ProductName | TotalAmount
```

Think of it like fast food — prepackaged, ready to serve.

---

## 🧠 Normalization — It’s All About **Order & Logic**

| Form    | Rule                           | Fix                                   |
| ------- | ------------------------------ | ------------------------------------- |
| **1NF** | No repeating groups            | Split multicolumn values into rows    |
| **2NF** | Full dependency on primary key | Break out partial dependencies        |
| **3NF** | No transitive dependencies     | Split columns that depend on non-keys |

🛍️ Example:
Your product has a `ProductionCountry` and a `ShortCode`. If `ShortCode` depends on `ProductionCountry`, **split it off**.

---

## 🌟 Denormalization = Weapon of OLAP Choice

### ⭐ Star Schema:

- One big **Fact** table (sales, orders)
- Many small **Dimension** tables (product, date, customer)

Simple, fast joins. Perfect for dashboards.

### ❄️ Snowflake Schema:

- Normalize the dimensions further.
- More joins, less redundancy. Good when dimension data is reused across multiple fact tables.

---

## 🔥 Data Types — Small Choices, Big Impact

| Type           | When to Use        | Why                |
| -------------- | ------------------ | ------------------ |
| `INT`          | Counting anything  | Small, fast        |
| `DECIMAL(p,s)` | Money or precision | No rounding errors |
| `VARCHAR(n)`   | Text               | Space-efficient    |
| `DATE`         | Dates              | Duh                |
| `BIT`          | Yes/No             | 1 bit storage      |

🧨 Don’t use `VARCHAR(50)` for everything. That’s rookie stuff.

---

## 🎯 Indexes: The Real MVP (If You Know How to Use Them)

| Type         | Best For               | Example                 |
| ------------ | ---------------------- | ----------------------- |
| Clustered    | Sorting, range queries | Primary key             |
| Nonclustered | Filters, lookups       | Email, Phone            |
| Filtered     | Rare subsets           | WHERE Status = ‘Active’ |
| Columnstore  | Data warehouses        | SUM(), AVG(), COUNT()   |

📦 Real Example:

```sql
CREATE NONCLUSTERED INDEX ix_products_productid
ON Products(ProductID)
INCLUDE (ProductName, Price);
```

Avoids **Key Lookup**, speeds up `SELECT`.

---

## 🧠 Columnstore Index = Cheat Code for Analytics

Imagine scanning **only the "SalesAmount"** column from 10M rows. That’s what Columnstore does.

- Compresses like a zip file.
- Reads in big blocks (900+ rows at once).
- Way faster for SUMs, GROUP BYs, etc.

---

## 🎬 Final Boss Summary

✅ **Design fits the use case** (OLTP ≠ OLAP)
✅ **Data types are lean & mean**
✅ **Indexes support your actual queries**
✅ **Normalize where needed, denormalize where it helps**
✅ **Use Columnstore for big analytics, not small transactions**

---
