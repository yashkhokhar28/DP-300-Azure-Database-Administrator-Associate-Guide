## 🔍 **What is SQL Injection?**

**SQL Injection (SQLi)** is a type of **attack** where an attacker tricks your application into running **unintended SQL commands**, often by providing specially crafted input.

➡️ Example:
A user enters something **malicious** instead of normal data, causing your database to execute harmful queries.

---

## 💣 **How Does It Work?**

Imagine a website has a query like this:

```sql
SELECT * FROM Orders WHERE OrderId=25
```

This runs fine when a user inputs an order ID like `25`.

But if someone enters this in the form instead:

```
25; DELETE FROM Orders;
```

The final SQL query becomes:

```sql
SELECT * FROM Orders WHERE OrderId=25; DELETE FROM Orders;
```

➡️ This query does **two things**:

1. Selects order 25 ✅
2. **Deletes the entire Orders table!** ❌

---

## ⚠️ **Why is SQL Injection Possible?**

- The app **directly inserts user input** into SQL statements.
- There is **no validation or escaping** of harmful input.
- Poor coding practices in dynamic SQL generation.

Even **legacy applications** and new apps without security focus are vulnerable.

---

## 🔧 **How Can You Prevent SQL Injection?**

### ✅ Best Practices:

| Technique                      | Description                                                          |
| ------------------------------ | -------------------------------------------------------------------- |
| Use **Parameterized Queries**  | Safest way. Example: `SELECT * FROM Orders WHERE OrderId = @OrderId` |
| Use **Stored Procedures**      | Precompiled SQL code, separating logic from user input               |
| Input Validation               | Check that input is what you expect (numbers, text length, etc.)     |
| Escape Special Characters      | Avoid letting quotes (`'`), semicolons (`;`), etc. break your query  |
| Use ORM Libraries              | Frameworks like Entity Framework, Hibernate prevent SQLi by default  |
| Minimal Privileges             | Give the database account **only the access it needs**               |
| Web Application Firewall (WAF) | Add an extra defense layer to block SQL injection patterns           |
| Use Advanced Threat Protection | Azure feature to monitor and alert suspicious SQL behavior           |

---

## 🔑 **Bad Security Example**

Trying to filter out keywords like "DELETE" in user input is **not enough**. Attackers can:

- Use binary data,
- Encode malicious queries,
- Bypass simple checks.

Example of encoded attack:

```sql
DECLARE @v VARCHAR(255)
SELECT @v = CAST(0x73705F68656C706462 AS VARCHAR(255))
EXEC (@v)
```

⚠️ This converts binary data into a SQL command and executes it.

---

## 🔐 **Key Takeaways**

| Concept           | Meaning                                                   |
| ----------------- | --------------------------------------------------------- |
| What is SQLi?     | Injecting harmful SQL commands into an application        |
| Common cause      | Directly using user input in SQL queries                  |
| Real-world impact | Data leaks, data loss, unauthorized access                |
| Main defense      | Parameterized queries, validation, and minimal privileges |
| Extra protection  | Azure Advanced Threat Protection, Web App Firewalls       |

---
