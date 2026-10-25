### ✅ **Azure SQL Ledger: Explained**

---

### 🔍 **What is Ledger in Azure SQL Database?**

Ledger in Azure SQL Database is a **cryptographic feature** that helps you prove that your **data hasn't been tampered with**.
It adds **tamper-evidence** to your database records, meaning you can show **auditors, business partners, or regulators** that your data is authentic and unmodified.

➡️ Think of it like a **blockchain-like feature inside your database** but **without the complexity of distributed blockchain networks**.

➡️ Available in **Azure SQL Database, Azure SQL Managed Instance and SQL Server 2022+**. 🔄

---

### 🔐 **How Does It Work?**

- Each **transaction** on the database is **hashed using SHA-256**, a secure cryptographic algorithm.
- These hashed records are **linked together**, like blocks in a blockchain.
- If someone modifies the data directly in the database **without using valid SQL operations**, the **cryptographic chain breaks**, making tampering **detectable**.
- You can provide a **cryptographic proof** showing whether your data has remained unchanged.

---

### 🧱 **Types of Ledger Tables**

| **Ledger Table Type**         | **Purpose**                                           | **Allows**                                                  |
| ----------------------------- | ----------------------------------------------------- | ----------------------------------------------------------- |
| **Updatable Ledger Tables**   | For **transactional systems**, system of record apps. | Inserts, updates, deletes.                                  |
| **Append-only Ledger Tables** | For **audit logs, accounting, security logs.**        | Only inserts. Updates and deletes are blocked at API level. |

---

#### ▶️ **Updatable Ledger Tables**

- Track full **history of changes** using system-versioning.
- Every update or delete is logged in a **history table**.
- Example use case: a banking application that updates account balances.

#### ▶️ **Append-only Ledger Tables**

- Prevents updates and deletes completely.
- Example use case: an **accounting journal** or an **event log**, where records shouldn't be changed.

---

### ⭐ **Benefits of Ledger**

| **Benefit**                | **What It Helps With**                                                    |
| -------------------------- | ------------------------------------------------------------------------- |
| **Simplifies Audits**      | Shows auditors that your data hasn’t been tampered with.                  |
| **Builds Trust**           | Ensures data integrity in multi-party business processes.                 |
| **Data Integrity**         | Ensures records are unchanged without slowing down your database queries. |
| **No Blockchain Overhead** | Offers blockchain-like proof without complex network setups.              |

---

### ⚙️ **How to Enable Ledger**

✅ **Enable during database creation only:**

- Go to the **Azure portal → Create SQL Database → Security tab → Ledger section.**
- Select **Enable for all future tables** if you want **every table in the database to be ledger-protected**.

🛡️ **Configuration Options:**

- Configure a **digest storage location** where Azure stores the cryptographic proofs: **Azure Blob Storage with an immutability policy** or **Azure Confidential Ledger** (automatic digest storage), or generate digests manually. 🔄
- Verify integrity with `sys.sp_verify_database_ledger_from_digest_storage` (or `sys.sp_verify_database_ledger` with a manually saved digest). 🔄
- Once the database is created, you **cannot turn the ledger feature on or off** for the whole database.
- You can also **enable ledger at the table level** using T-SQL:
  Example:

  ```sql
  CREATE TABLE dbo.MyLedgerTable (
      ID INT PRIMARY KEY,
      Description NVARCHAR(100)
  ) WITH (SYSTEM_VERSIONING = ON, LEDGER = ON);   -- 🔄 updatable ledger table; append-only: WITH (LEDGER = ON (APPEND_ONLY = ON))
  ```

---

### ⚠️ **Important Notes**

- **Tamper-evidence only:** Ledger **doesn't prevent tampering**, but **detects it**.
- The data is stored **on-chain in the database itself**, but digests of the cryptographic chain can be optionally stored externally.
- You need to plan for **ledger use at the start of your project**, because enabling it later is not supported at the database level.

---

### ✅ **When Should You Use Ledger?**

✔️ When your organization requires **auditable proof of data integrity**.
✔️ For **financial records, compliance logs, auditing tables**, and other mission-critical data.
✔️ To avoid using a full **blockchain network**, but still provide **tamper evidence**.

---
