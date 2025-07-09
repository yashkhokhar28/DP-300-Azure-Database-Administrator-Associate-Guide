## 🧱 What is Normalization?

**Normalization** is a process in database design that structures data to:

- **Reduce redundancy**
- **Improve data integrity**
- **Avoid update anomalies**

The idea is:

> “One fact in one place.”

Each table should represent one entity (e.g., Customer, Product), and only data **directly related** to that entity should be in the table.

---

## 🔢 1st Normal Form (1NF)

### ✅ **Rules:**

1. No **repeating groups** or arrays.
2. Each **column holds atomic values** (not lists).
3. Each **row is unique**, identified by a **primary key**.

---

### ❌ Example: _Not in 1NF_

```plaintext
ProductID | Color1 | Color2 | Color3
-----------------------------------
1         | Red    | Green  | Yellow
2         | Yellow | NULL   | NULL
```

- Multiple colors in different columns → **Repeating groups**

---

### ✅ Converted to 1NF:

```plaintext
ProductID | Color
------------------
1         | Red
1         | Green
1         | Yellow
2         | Yellow
```

- Each color is in a separate row — **atomic**, and can scale easily.

---

## 🧮 2nd Normal Form (2NF)

### ✅ **Rules (in addition to 1NF):**

- For tables with **composite primary keys**, **all non-key columns** must depend on **the whole key**, not just part of it.

---

### ❌ Example: _Not in 2NF_

```plaintext
ProductID | Color  | Price
---------------------------
1         | Red    | 15.95
1         | Green  | 15.95
```

- Composite key: `ProductID + Color`
- But `Price` only depends on `ProductID`, not `Color` → violates 2NF.

---

### ✅ Fix: Split into two tables

```plaintext
-- ProductColor Table (Composite Key)
ProductID | Color
------------------
1         | Red
1         | Green

-- Product Table
ProductID | Price
------------------
1         | 15.95
```

---

## 🧠 3rd Normal Form (3NF)

### ✅ **Rules (in addition to 2NF):**

- No **transitive dependencies** (non-key columns depending on other non-key columns).

---

### ❌ Example: _Not in 3NF_

```plaintext
ProductID | ProductName | ProductionCountry | ShortLocation
------------------------------------------------------------
1         | Widget      | United States     | US
2         | Foop        | United Kingdom    | UK
```

- `ShortLocation` depends on `ProductionCountry`, not `ProductID` → violates 3NF.

---

### ✅ Fix: Move ShortLocation to separate table

```plaintext
-- ProductInfo Table
ProductID | ProductName | ProductionCountry
--------------------------------------------
1         | Widget      | United States

-- CountryCodes Table
ProductionCountry    | ShortLocation
---------------------|--------------
United States        | US
```

---

## 🔄 Denormalization

Normalization ensures data integrity, but it can **slow down performance** in **read-heavy workloads**, due to too many joins.

So, in **data warehouses**, it's often better to:

- **Combine tables**
- **Store redundant data**
- **Reduce joins**

This is called **denormalization**.

---

## ⭐ Star Schema (Data Warehouse)

- Central **fact table** (e.g., sales)
- Connected to multiple **dimension tables** (e.g., product, customer)

### Example:

**FactResellerSales**

```plaintext
SalesID | DateKey | ProductKey | Revenue
----------------------------------------
1       | 20240701| 1001       | 1200
```

**DimProduct**

```plaintext
ProductKey | ProductName | Category
-----------------------------------
1001       | Laptop      | Electronics
```

Simple joins, fast queries.

---

## ❄️ Snowflake Schema

A more **normalized version** of the star schema:

- Dimensions are **split** into multiple related tables.

### Example:

**DimProduct**

```plaintext
ProductKey | ProductName | SubcategoryKey
------------------------------------------
1001       | Laptop      | 200
```

**DimProductSubcategory**

```plaintext
SubcategoryKey | SubcategoryName | CategoryKey
-----------------------------------------------
200            | Computers        | 50
```

**DimProductCategory**

```plaintext
CategoryKey | CategoryName
---------------------------
50          | Electronics
```

- **More joins**, but saves storage.
- Good when dimension data is shared or repetitive.

---

## 🎯 Summary

| Concept              | Goal                            | Example Fix                                  |
| -------------------- | ------------------------------- | -------------------------------------------- |
| 1NF (No repeating)   | Atomic values                   | Move Color1, Color2 → multiple rows          |
| 2NF (No partial dep) | Each column depends on full key | Move Price from ProductColor → Product table |
| 3NF (No transitive)  | No column depends on non-key    | Move ShortLocation to Country table          |
| Denormalization      | Faster reads (OLAP)             | Store redundant data, fewer joins            |
| Star Schema          | Simple read model               | Sales + Products, Customers, Time            |
| Snowflake Schema     | Normalized version of Star      | Product split into Category/Subcategory      |

---
