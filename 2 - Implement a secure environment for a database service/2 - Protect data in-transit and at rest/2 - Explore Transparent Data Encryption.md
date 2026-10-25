## ✅ **What is Transparent Data Encryption (TDE)?**

TDE **encrypts the entire database at the storage level (data pages on disk)**. Data is:

- **Encrypted when written to disk (at the page level)**
- **Decrypted when loaded into memory**

It protects **data at rest**, meaning:

- Files on disk, backups, and transaction logs are encrypted.
- It **does not encrypt at the column or table level**. Users with read access can still query data normally.

---

## ✅ **Why Use TDE?**

- Prevents unauthorized users from accessing database files if they are copied to another server.
- Meets compliance standards (GDPR, HIPAA, etc.).
- Protects backups without changing application code.
- Supports **AES_128 / AES_192 / AES_256** (Azure SQL uses AES-256). `TRIPLE_DES_3KEY` (3DES) is deprecated since SQL Server 2016. 🔄

---

## ✅ **When is TDE Enabled Automatically?**

| Platform                   | Auto-enabled for new databases after |
| -------------------------- | ------------------------------------ |
| Azure SQL Database         | May 2017                             |
| Azure SQL Managed Instance | February 2019                        |

Databases created **before these dates must have TDE enabled manually.**

---

## ✅ **How TDE Works in Azure SQL Database**

- By default, Azure uses a **service-managed key** (Microsoft generates and manages the encryption key).
- You can bring your own key (**BYOK**, customer-managed) using **Azure Key Vault (AKV).**
- If the BYOK is deleted or disabled, access to the database is blocked.

➡️ **Configure TDE in Azure Portal:**

- Open your database → **Transparent Data Encryption** → Enable or configure keys.

---

## ✅ **Enabling TDE on SQL Server (on-prem or Azure VM)**

You run the following T-SQL commands:

```sql
USE master;
GO

-- 1. Create a master key (protected by a password)
CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<your-pwd>';
GO

-- 2. Create a certificate to protect the database encryption key
CREATE CERTIFICATE MyServerCert
    WITH SUBJECT = 'TDEDemo_Certificate';
GO

-- 3. Create the encryption key in the user database
USE [TDE_Demo];
GO

CREATE DATABASE ENCRYPTION KEY
    WITH ALGORITHM = AES_256
    ENCRYPTION BY SERVER CERTIFICATE MyServerCert;
GO

-- 4. Turn on encryption
ALTER DATABASE TDE_Demo SET ENCRYPTION ON;
GO
```

⚠️ **Important:** Backup your certificate and master key securely. Without them, you can’t restore encrypted backups.

---

## ✅ **How Long Does Encryption Take?**

Encrypting a large database takes time because SQL Server must:

- Read each page,
- Encrypt it,
- Write it back to disk.

The process runs in the background with low priority to avoid impacting performance.

---

## ✅ **Certificates in Availability Groups**

If you use **Always On Availability Groups**, the **certificate must be copied to all replica servers**, or the databases won’t be readable after a failover.

---

## ✅ **Customer-Managed Keys (BYOK) with Azure Key Vault**

Instead of using Microsoft's service-managed key, you can use your **own key stored in Azure Key Vault**, providing:

- Full control over who can use the key.
- Better security separation between key management and data access.
- Key rotation and revocation options.

Azure supports:

- **User-assigned managed identities (UMI)**: Pre-configure database servers with access to Key Vault.
- **Automatic key rotation:** the server (or database) continuously checks Key Vault for a new version of the key and rotates the TDE protector to it **within 24 hours**. Only the DEK is re-encrypted, online. Portal: **Auto-rotate key** checkbox; PowerShell `-AutoRotationEnabled`, CLI `--auto-rotation-enabled`. 🔄
- **Database-level CMK** (Azure SQL Database only, not MI): each database can have its own TDE protector key, accessed with a **user-assigned managed identity** (system-assigned isn't supported at database level). Supports cross-tenant keys and database-level auto-rotation (`Set-AzSqlDatabase -EncryptionProtectorAutoRotation`). 🔄

---

## ✅ **Azure Disk Encryption**

For **Azure Virtual Machines running SQL Server**, use **Azure Disk Encryption (ADE)** to add an extra layer of encryption at the disk level.

- This protects the entire disk, even before SQL Server loads data into memory.
- Used alongside TDE for layered security.
- ⚠️ **Azure Disk Encryption retires 15 Sep 2028.** Microsoft recommends **encryption at host** for new VMs; ADE-enabled VMs must migrate before that date. 🔄

---

## ✅ **Key Benefits of TDE**

| Benefit                              | Description                                                |
| ------------------------------------ | ---------------------------------------------------------- |
| Encrypts all data at rest            | Data is protected in database files and backups.           |
| Transparent to applications          | No code changes required.                                  |
| Compliant with industry regulations  | Helps meet GDPR, HIPAA, PCI-DSS, etc.                      |
| BYOK and Azure Key Vault integration | Gives you full control over your encryption keys.          |
| Automatic on new Azure SQL databases | Already enabled on most new databases, no action required. |

---

## 🔑 **Summary**

- TDE encrypts entire databases, protecting data **at rest**.
- It’s easy to enable and **transparent to applications**.
- In Azure, you can let Microsoft manage the keys, or use your own keys in Azure Key Vault for greater control.

---
