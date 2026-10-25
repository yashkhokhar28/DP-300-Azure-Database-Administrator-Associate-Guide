## 🔐 **What is Object Encryption in SQL Server?**

Object encryption means **encrypting sensitive data inside database columns**, so that even **privileged users like DBAs or cloud admins cannot read it**.

The main feature for this is **Always Encrypted**, which protects data from being visible **even inside the SQL Server** itself.

---

## ✅ **Always Encrypted Overview**

- **Protects sensitive column data** (like credit card numbers, SSNs).
- Data is encrypted **at the client-side** before being sent to SQL Server.
- SQL Server **never sees the plaintext data**—it only stores the encrypted data.
- Even admins with full database access can't view the data unless they have the right encryption keys.

---

## 🔑 **How Always Encrypted Works**

Always Encrypted uses **two types of keys**:

| Key Type                        | Purpose                                                                                                 |
| ------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Column Master Key (CMK)**     | Protects the column encryption keys; stored securely (Azure Key Vault, Windows Certificate Store, etc.) |
| **Column Encryption Key (CEK)** | Actually encrypts the data in the column                                                                |

The client driver reads the **encrypted CEK** from the database and decrypts it with the **CMK**. Only the application (not SQL Server) can access the CMK. 🔄

---

## 📄 **Example Scenarios for Always Encrypted**

| Scenario                         | Benefit                                                              |
| -------------------------------- | -------------------------------------------------------------------- |
| On-prem app & database           | Protect data from on-prem DBAs/admins.                               |
| On-prem app & Azure SQL Database | Protect data from Microsoft cloud administrators.                    |
| App and database both in Azure   | Still encrypts data, though Azure platform admins could manage keys. |

---

## 🔁 **Encryption Types in Always Encrypted**

| Encryption Type   | Features                                                                                        | Limitations                          |
| ----------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------ |
| **Deterministic** | Same plaintext always encrypts to the same ciphertext. Supports **joins, where, and group by**. | Predictable, vulnerable to guessing. |
| **Randomized**    | Same plaintext encrypts differently each time. More secure.                                     | No joins, comparisons, or indexing.  |

---

## 🛡️ **Secure Enclaves (Advanced Feature)**

Secure enclaves are **isolated memory areas** inside the SQL Server process that allow:

- Secure execution of some operations (like comparisons or filtering) on encrypted data.
- SQL Server still **cannot see inside the enclave**.

With secure enclaves, you can:

- Perform operations on **randomized encrypted columns**.
- Run **pattern matching, comparisons, and indexing** on encrypted data.

⚠️ Secure enclaves use **VBS enclaves** (software-based, no special hardware; SQL Server 2019+ on Windows and Azure SQL Database) or **Intel SGX enclaves** (Azure SQL Database DC-series only, end of support 31 Oct 2027); not supported on Managed Instance. ➡️ See **8 - [NEW] Always Encrypted with secure enclaves.md** for details.

---

## 🛠️ **How to Enable Always Encrypted**

You can enable encryption using:

- **SQL Server Management Studio (SSMS) Encryption Wizard**
- **T-SQL commands**

Main steps:

1. Select columns to encrypt.
2. Choose encryption type (Deterministic or Randomized).
3. Configure key storage (Azure Key Vault, Windows Cert Store, etc.).
4. Encrypt the data.

Example T-SQL:

```sql
CREATE COLUMN MASTER KEY MyCMK
WITH (
    KEY_STORE_PROVIDER_NAME = 'AZURE_KEY_VAULT',
    KEY_PATH = 'https://<key-vault-name>.vault.azure.net/keys/<key-name>'
);

CREATE COLUMN ENCRYPTION KEY MyCEK
WITH VALUES (
    COLUMN_MASTER_KEY = MyCMK,
    ALGORITHM = 'RSA_OAEP',
    ENCRYPTED_VALUE = 0x1234ABCD...
);
```

---

## 💻 **How Applications Access Encrypted Columns**

- Application must use an **Always Encrypted-compatible driver** (e.g., ADO.NET, ODBC).
- The **connection string** must include:

  ```
  Column Encryption Setting=Enabled
  ```

- App fetches the encryption keys from Azure Key Vault (or another store).
- Decryption happens on the client side.

---

## 🔑 **Key Security Tips**

✔️ Store your **master keys in secure stores** like Azure Key Vault or HSMs.
❌ Don’t store encryption keys on the SQL Server itself.
✔️ Backup your master keys. If lost, you **cannot decrypt your data**.

---

## ✅ **Summary of Always Encrypted vs TDE**

| Feature                           | Always Encrypted | Transparent Data Encryption (TDE) |
| --------------------------------- | ---------------- | --------------------------------- |
| Scope                             | Specific columns | Whole database                    |
| Who sees data unencrypted         | Client only      | SQL Server + Client               |
| Protects from DBAs                | Yes              | No                                |
| Useful for compliance (GDPR, PCI) | Yes              | Yes                               |

---
