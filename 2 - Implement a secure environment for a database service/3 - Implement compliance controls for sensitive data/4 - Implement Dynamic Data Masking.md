## ✅ **What is Dynamic Data Masking (DDM)?**

**Dynamic Data Masking** is a security feature that hides sensitive data at the **presentation layer**. This means:

- **The actual data remains unchanged in the database.**
- Only the **displayed results** for unauthorized users are masked.
- Users with proper privileges (or the `UNMASK` permission) can see the real data.

---

## 🔍 **How Does It Work?**

- Suppose a user runs a query like:
  `SELECT PhoneNumber FROM Employees;`

  - If the user has **only SELECT permission**, they will see something like: `XXXXXX1234`.
  - If the user also has the **UNMASK permission**, they will see the actual phone number, like `9876541234`.

- **DDM applies masking rules only to query results**.

- It’s configured per **column**, so you can selectively mask only sensitive fields like email, phone number, etc.

---

## ⚙️ **How to Configure DDM?**

You can configure it:

- In the **Azure portal** → Go to your database → **Security → Dynamic Data Masking**
- Using **T-SQL commands**

Example T-SQL:

```sql
ALTER TABLE [Employee] ALTER COLUMN [Email]
ADD MASKED WITH (FUNCTION = 'email()');
```

---

## 🔑 **Masking Functions Available**

| **Masking Function** | **Purpose**                                                       | **T-SQL Example**                                                                                          |
| -------------------- | ----------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **default()**        | Hides all data (`XXXX`, `0`, `01.01.1900`)                        | `ALTER TABLE [Customer] ALTER COLUMN Address ADD MASKED WITH (FUNCTION = 'default()')`                     |
| **partial()**        | Shows part of the data, masks the rest. Useful for phone numbers. | `ALTER TABLE [Customer] ALTER COLUMN PhoneNumber ADD MASKED WITH (FUNCTION = 'partial(0,"XXXX-XXXX-",4)')` |
| **email()**          | Masks emails (`aXXXX@XXXX.com`)                                   | `ALTER TABLE [Customer] ALTER COLUMN Email ADD MASKED WITH (FUNCTION = 'email()')`                         |
| **random()**         | Shows random numbers instead of actual numbers.                   | `ALTER TABLE [Customer] ALTER COLUMN Month ADD MASKED WITH (FUNCTION = 'random(1,12)')`                    |
| **custom string**    | Fully customizable masking.                                       | `ALTER TABLE [Customer] ALTER COLUMN PhoneNumber ADD MASKED WITH (FUNCTION = 'partial(1,"XXXX",2)')`       |

---

## 🔒 **How to View Unmasked Data?**

- You must explicitly grant the **UNMASK** permission:

```sql
GRANT UNMASK TO [UserName];
```

---

## ⚠️ **Limitations and Considerations**

- **Masking is only at the query output**. The actual data in the table is **not encrypted**.
- Advanced users might infer sensitive data by running creative queries (**inference attacks**).
  → To mitigate this, combine DDM with other security features like:

  - **Auditing**
  - **Encryption**
  - **Row-Level Security**

---

## ✅ **Best Use Cases for Dynamic Data Masking**

✔️ Hide sensitive data from application users.
✔️ Share databases with vendors without exposing confidential data.
✔️ Protect data in non-production environments (Dev, Test).
✔️ Allow customer service to see only the last 4 digits of phone numbers or credit cards.

---

## 📤 **Behavior During Export and Import**

- If a user without `UNMASK` permission runs **Export or Copy commands**, the exported data will remain masked.

  - Example:

  ```sql
  SELECT INTO BackupTable FROM SensitiveTable;
  ```

  → `BackupTable` will contain the **masked data**.

---

## 📝 **Summary**

- Dynamic Data Masking **simplifies securing sensitive columns** without changing the database structure.
- Supports several masking patterns (**default, partial, email, random**).
- Should be used **alongside other security controls** for stronger protection.
- Quick to implement, minimal application changes required.

---
