### ✅ **Row-Level Security (RLS) in SQL Server & Azure SQL: Explained**

---

### 🔍 **What is Row-Level Security?**

**Row-Level Security (RLS)** restricts data access at the **row level**, meaning:

- Users **only see the rows they’re permitted to see**.
- It’s like **applying an automatic WHERE clause** to your queries.
- The filtering logic is enforced directly in the **database**, not in your application.

➡️ Example:
If a user from **Tenant1** queries the `Sales` table, they only see rows where `TenantName = 'Tenant1'`.

---

### 🔑 **How Does RLS Work?**

RLS works by:

1. Creating a **table-valued function (TVF)** that defines the filtering condition.
2. Creating a **security policy** that applies this function to the table.

The SQL Server engine automatically calls the function whenever someone queries the table, ensuring **consistent filtering**, regardless of how the query is written.

---

### 🔨 **Two Types of Security Policies in RLS**

| **Type**              | **Use Case**                                                       | **Affects Operations** |
| --------------------- | ------------------------------------------------------------------ | ---------------------- |
| **Filter predicates** | Hides rows that the user shouldn't see.                            | SELECT, UPDATE, DELETE |
| **Block predicates**  | Stops users from inserting, updating, or deleting restricted rows. | INSERT, UPDATE, DELETE |

---

### 🚀 **Example Scenario: Multi-Tenant Data Isolation**

➡️ **Step 1:** Create the `Sales` table and add some data:

```sql
CREATE TABLE [Sales] (SalesID INT, ProductID INT, TenantName NVARCHAR(10), OrderQtd INT, UnitPrice MONEY);
-- Insert rows for multiple tenants
```

➡️ **Step 2:** Create user accounts, one for each tenant:

```sql
CREATE USER [Tenant1] WITH PASSWORD = '...';
GRANT SELECT ON [Sales] TO [Tenant1];
```

➡️ **Step 3:** Create an **inline table-valued function (TVF)** to control access:

```sql
CREATE FUNCTION sec.tvf_SecurityPredicatebyTenant(@TenantName AS NVARCHAR(10))
RETURNS TABLE
WITH SCHEMABINDING   -- 🔄 required: security policies default to SCHEMABINDING = ON
AS
RETURN SELECT 1 AS result
WHERE @TenantName = USER_NAME() OR USER_NAME() = 'TenantAdmin';
```

This function lets users view rows where their username matches the `TenantName` column, or if they are an admin.

➡️ **Step 4:** Create the **security policy**:

```sql
CREATE SECURITY POLICY sec.SalesPolicy
ADD FILTER PREDICATE sec.tvf_SecurityPredicatebyTenant(TenantName) ON [dbo].[Sales]
WITH (STATE = ON);
```

➡️ **Step 5:** Test access:

```sql
EXECUTE AS USER = 'Tenant1';
SELECT * FROM Sales;
REVERT;
```

- Tenant1 only sees their data.
- `TenantAdmin` sees all data.

---

### ⚠️ **Limitations & Security Considerations**

- RLS **does not encrypt data**, it just controls who can query what.
- Beware of **side-channel attacks** (e.g., using error-based queries like divide-by-zero) to infer whether rows exist.
- Limit user query abilities (e.g., prevent ad hoc query execution).

---

### 📄 **Best Practices for RLS**

✔️ Place your predicate functions and security policies in a **separate schema** (for clarity and control).
✔️ Keep your predicate functions **simple and efficient**—avoid heavy joins and conversions.
✔️ Use RLS in combination with other security measures: **auditing**, **dynamic data masking**, and **encryption**.

---

### 🎯 **Use Cases for Row-Level Security**

- Multi-tenant applications: Each customer sees only their data.
- Department-level access: HR, Sales, and Finance see different employee data.
- Compliance-driven data segregation.

---
