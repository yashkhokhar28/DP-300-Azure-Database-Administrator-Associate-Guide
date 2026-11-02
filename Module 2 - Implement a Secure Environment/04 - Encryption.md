# Encryption

> **Exam skills:** Implement transparent data encryption (TDE), Implement object-level encryption, Implement Always Encrypted, Implement Always Encrypted with secure enclaves
> **Reading time:** ~20 min

## The big idea
Encryption scrambles data so only someone with the right **key** can read it. There are three places to protect data:
- **At rest** (stored on disk) → **TDE**.
- **In use, for chosen columns** (even the DBA must not see it) → **Always Encrypted**.
- **In transit** (moving over the network) → **TLS**.

Think of a bank. TDE is the locked vault. Always Encrypted is a sealed envelope that only the customer can open, even the bank staff cannot. TLS is the armoured van that moves money between branches.

## Transparent Data Encryption (TDE)
**TDE** encrypts the whole database **on disk**. It encrypts data files, log files and **backups**.
- Data is encrypted when written to disk (page by page) and decrypted when read into memory.
- It is **transparent**: apps need **no code changes**.
- Anyone with SELECT rights still sees normal data. TDE does **not** hide data from users or DBAs.
- It protects against someone **stealing the files or backups** and opening them on another server.
- Algorithms: AES_128, AES_192, AES_256. Azure SQL uses **AES-256**. TRIPLE_DES_3KEY (3DES) is deprecated since SQL Server 2016.

**Example:** A thief copies a backup file of a hospital database. Without the TDE key, the file is unreadable on any other server.

### TDE in Azure SQL
- **On by default** for new databases: Azure SQL Database since **May 2017**, Managed Instance since **February 2019**. Older databases must be turned on by hand.
- Default key: **service-managed** (Microsoft creates and rotates it).
- Or **customer-managed key (CMK)**, also called **BYOK** (bring your own key). The key lives in **Azure Key Vault** (see below).
- Portal: database → **Transparent data encryption**.
- Encrypting a big database takes time. It runs in the background at low priority.

### TDE on SQL Server (VM or on-premises)
You set it up yourself with T-SQL. The chain of keys is:
**Service master key** (made at SQL setup) → protects the **database master key** → protects a **certificate** → protects the **database encryption key (DEK)** → encrypts the data.

```sql
USE master;
CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<pwd>';
CREATE CERTIFICATE MyServerCert WITH SUBJECT = 'TDE cert';
USE TDE_Demo;
CREATE DATABASE ENCRYPTION KEY WITH ALGORITHM = AES_256
    ENCRYPTION BY SERVER CERTIFICATE MyServerCert;
ALTER DATABASE TDE_Demo SET ENCRYPTION ON;
```

- **Back up the certificate and its private key.** Without them you cannot restore the encrypted backups anywhere.
- **Always On availability groups:** copy the certificate to **every replica**. If not, the database cannot be read after a failover.

### Disk encryption on Azure VMs
For SQL Server on a VM you can also encrypt the whole disk, as an extra layer under TDE.
- **Azure Disk Encryption (ADE)** retires on **15 Sep 2028**. ADE-enabled VMs must move off it before then.
- Microsoft recommends **encryption at host** for new VMs.

## Azure Key Vault and customer-managed TDE keys
**Azure Key Vault** is a safe cloud store for **secrets** (passwords), **keys** and **certificates**.
- Keys are protected by software (**Standard** tier) or by **HSMs** (hardware security modules: special tamper-proof devices) in the **Premium** tier or **Managed HSM**.
- It has its **own access control**. Even a subscription admin cannot read keys unless given access.
- It logs who used which key, for audits.

### Customer-managed key (CMK) for TDE in Azure SQL
- The TDE key (called the **TDE protector**) sits in your Key Vault. You control it.
- If you **delete or disable** the key, the database becomes **inaccessible**. That is your "kill switch".
- The server reaches Key Vault with a **managed identity**. A **user-assigned managed identity** can be set up before the server exists.
- **Automatic key rotation:** the server checks Key Vault for a new key version and switches to it **within 24 hours**. Only the DEK is re-encrypted, online. Portal: **Auto-rotate key** checkbox; PowerShell `-AutoRotationEnabled`; CLI `--auto-rotation-enabled`.
- **Database-level CMK** (Azure SQL Database only, not MI): each database can have its own key. It needs a **user-assigned** managed identity (system-assigned is not supported at database level). It supports keys in another tenant and per-database auto-rotation.

**Example:** A bank must prove that it, not Microsoft, controls its encryption keys. It stores the TDE key in its own Key Vault. If the contract ends, it disables the key and the data is locked.

### Key Vault with SQL Server on a VM
SQL Server on a VM (or on-premises) can keep its TDE, backup encryption and cell-level encryption keys in Key Vault. This uses **EKM** (Extensible Key Management: a plug-in that lets SQL Server use outside key stores). Steps in short:
1. Create a Key Vault and a key. Register an app in Entra ID.
2. Give the app the **Key Vault Crypto Service Encryption User** role (or old access-policy rights: get, list, wrapKey, unwrapKey).
3. Install the **SQL Server Connector for Azure Key Vault**, enable EKM, create the cryptographic provider.
4. Create a **credential** and an **asymmetric key** from the provider.

```sql
CREATE CREDENTIAL sysadmin_ekm_cred
WITH IDENTITY = 'mykeyvault',                       -- Key Vault name
SECRET = '<ClientIdWithoutHyphens><ClientSecret>'
FOR CRYPTOGRAPHIC PROVIDER AzureKeyVault_EKM;

CREATE ASYMMETRIC KEY MyKey FROM PROVIDER [AzureKeyVault_EKM]
WITH PROVIDER_KEY_NAME = 'MyAzureKey', CREATION_DISPOSITION = OPEN_EXISTING;
```

- **SQL Server 2022 CU17+ on an Azure VM** (or SQL Server 2025 enabled by Azure Arc) can use a **managed identity** instead of the app registration. On-premises SQL Server still needs a service principal.

## Object-level (cell-level) encryption
You can encrypt single values or columns inside SQL Server using **symmetric keys** and certificates. The app (or T-SQL) encrypts and decrypts with functions on the server.

**Example:** An HR database encrypts the salary column with a symmetric key. Only payroll staff who can open the key can decrypt it.

The downside: the **server** holds the keys and sees the plain values while decrypting. A powerful DBA can still read them. To keep data hidden even from the DBA, use **Always Encrypted**.

## Always Encrypted
**Always Encrypted** encrypts chosen columns **on the client side** (inside the app's database driver).
- SQL Server **never sees the plain data** or the keys. It only stores encrypted bytes.
- DBAs, cloud admins and Microsoft staff **cannot** read the values.
- Good for credit card numbers, national ID numbers, health data.

### Two keys

| Key | What it does | Where it lives |
|-----|-------------|----------------|
| **Column master key (CMK)** | Protects the column encryption keys | Outside SQL: **Azure Key Vault**, Windows Certificate Store, or an HSM |
| **Column encryption key (CEK)** | Encrypts the column data | Stored in the database, but **encrypted** by the CMK |

The client driver reads the encrypted CEK from the database, decrypts it with the CMK, then decrypts the data. Only the app can reach the CMK.

Note: the Always Encrypted acronym CMK (column master key) is different from TDE's CMK (customer-managed key). Read the question carefully.

### Two encryption types

| Type | How it works | Can query? |
|------|--------------|-----------|
| **Deterministic** | Same value → always same encrypted value | Equality: `=`, joins, GROUP BY, indexes. Easier to guess. |
| **Randomized** | Same value → different encrypted value each time | More secure, but no searching or joining (unless you use enclaves) |

**Example:** A clinic encrypts `NationalID` with **deterministic** so staff can look up a patient by exact ID. It encrypts `Diagnosis` with **randomized** because nobody searches on it.

### How to set it up
- Use the **Always Encrypted wizard** in SSMS (pick columns, pick type, pick key store), or PowerShell, or T-SQL for the key metadata:

```sql
CREATE COLUMN MASTER KEY MyCMK
WITH (KEY_STORE_PROVIDER_NAME = 'AZURE_KEY_VAULT',
      KEY_PATH = 'https://<vault>.vault.azure.net/keys/<key>');
CREATE COLUMN ENCRYPTION KEY MyCEK
WITH VALUES (COLUMN_MASTER_KEY = MyCMK, ALGORITHM = 'RSA_OAEP',
             ENCRYPTED_VALUE = 0x01...);
```

- The app must use an Always Encrypted-capable driver (e.g. ADO.NET / Microsoft.Data.SqlClient, ODBC, JDBC) with **`Column Encryption Setting=Enabled`** in the connection string.
- Never store the CMK on the SQL Server machine. **Back up the CMK**: if it is lost, the data is lost.

### Where it helps

| Setup | What it protects against |
|-------|--------------------------|
| On-premises app + on-premises database | On-premises DBAs |
| On-premises app + Azure SQL | Cloud admins (keys stay on-premises) |
| App and database both in Azure | Still encrypted, but Azure admins could manage the keys |

## Always Encrypted with secure enclaves
Plain Always Encrypted has a limit. The server cannot work on the plain data, so it can only do **equality on deterministic** columns. Range searches, LIKE, sorting and even first-time encryption must happen in the client app. For a big table, that means pulling all data out and pushing it back.

A **secure enclave** fixes this. It is a **protected, locked-off area of memory inside the SQL Server process**. Nobody outside can look in: not the DBA, not the OS admin, not a debugger.
- The client driver sends the column keys **into the enclave** over a secure channel.
- SQL Server hands the encrypted work to the enclave. The enclave decrypts and computes **only inside itself**.

**Example:** An HR app must find employees with salary between 40,000 and 45,000. Salary is encrypted with randomized encryption. With an enclave, SQL Server can run the `BETWEEN` inside the enclave. The DBA still never sees a salary.

### What enclaves add

| Ability | Plain Always Encrypted | With secure enclaves |
|---------|------------------------|----------------------|
| `=` | Deterministic only | Deterministic **and randomized** |
| `<`, `>`, `BETWEEN`, `IN`, `LIKE`, `DISTINCT` | No | Yes (randomized columns) |
| Joins | Deterministic equality joins | Yes (SQL Server 2019: nested loop joins only) |
| `ORDER BY`, `GROUP BY` | No | Yes in Azure SQL Database and SQL Server 2022+ (not 2019) |
| Indexes on randomized columns | No | **Nonclustered** only |
| Encrypt, rotate keys, decrypt | Client tool must move the data | **In place** with `ALTER TABLE ... ALTER COLUMN` |

- Rich queries (range, LIKE, sort) need **randomized** encryption. A deterministic enclave-enabled column still gets **equality only**.
- **In-place encryption** works for both types, if all keys are enclave-enabled.
- Text columns in enclave queries need a **`_BIN2` collation** (Azure SQL Database and SQL Server 2022 also accept **UTF-8** collations).

### Two enclave types: VBS and Intel SGX

| | **VBS enclaves** | **Intel SGX enclaves** |
|--|------------------|------------------------|
| What it is | **Software**-based, uses the Windows hypervisor (virtualization-based security). No special hardware. | **Hardware**-based, built into Intel CPUs |
| SQL Server 2019 / 2022 / 2025 on Windows (on-premises or VM) | **Yes – the only type** | No |
| Azure SQL Database | Yes, on **any** tier and model (vCore, DTU, serverless, elastic pools) **except DC-series** | Only on **DC-series** hardware (vCore) |
| Protects against | Attacks inside the VM / guest OS (e.g. DBA, memory dumps) | Attacks from guest OS **and** host OS |
| Status | Current, **recommended** | **End of support 31 Oct 2027** |

- **Managed Instance: no secure enclaves.** Plain Always Encrypted works on MI.
- **SQL Server on Linux: no secure enclaves.**
- After 31 Oct 2027, Azure moves databases still on DC-series to standard-series and **turns on VBS enclaves**.
- DC-series facts, if asked: uses **physical** cores, max **40** cores, **no serverless**, few regions.

### Attestation
**Attestation** means the client checks that the enclave is genuine and untampered **before** sending it keys.

| Product | Enclave | Attestation |
|---------|---------|-------------|
| SQL Server 2019+ | VBS | **Host Guardian Service (HGS)** or **None** |
| Azure SQL Database | SGX (DC-series) | **Microsoft Azure Attestation (MAA)** – required |
| Azure SQL Database | VBS | **None** only (attestation not supported) |

- **HGS** is a Windows Server role. The SQL Server host is registered with it.
- **MAA**: create an attestation provider, set its policy, give the server access, share the **attestation URL** with the app team.
- **None** is quickest to set up, but the client cannot detect a tampered enclave.
- Keep roles apart: the attestation admin should not be the DBA.

### Enclave-enabled keys
- **Enclave-enabled CMK:** has the **`ENCLAVE_COMPUTATIONS`** property and a signature. In SSMS, tick **Allow enclave computations** when you create the CMK.
- **Enclave-enabled CEK:** any CEK protected by an enclave-enabled CMK. Only these can be used in the enclave.
- Enclave-enabled CMKs can live only in **Windows Certificate Store** or **Azure Key Vault**.

### Turn on the enclave
**Azure SQL Database (VBS)** – off by default. It sets **`preferredEnclaveType = VBS`** on the database or elastic pool.
- Portal: when creating, **Security** tab → **Always Encrypted** → **Enable secure enclaves = ON**. For an existing database: **Security → Data Encryption → Always Encrypted** tab. For an existing pool: **Settings → Configuration → Always Encrypted**.
- PowerShell: `Set-AzSqlDatabase ... -PreferredEnclaveType VBS`. CLI: `az sql db update ... --preferred-enclave-type VBS`.
- Databases added to a VBS pool **inherit** VBS. Adding a VBS database to a non-VBS pool is **not supported**.

**Azure SQL Database (SGX):** choose **DC-series** hardware and set up **MAA**.

**SQL Server 2019+ (VBS):** needs Windows Server 2019+ (or Windows 11 24H2+), CPU virtualization, and VBS running. In Azure, use a **Generation 2 VM**.

```sql
EXEC sp_configure 'column encryption enclave type', 1;   -- 0 = none, 1 = VBS
RECONFIGURE;
-- Restart the SQL Server instance, then check:
SELECT * FROM sys.dm_column_encryption_enclave;   -- one row = enclave ready
```

### Using it
- Connection string: `Column Encryption Setting=Enabled;Attestation Protocol=None;` (or `HGS` / `AAS` plus `Enclave Attestation Url`). If **Attestation Protocol is missing, enclaves are off**.
- SSMS: Connect → Options → **Always Encrypted** tab → Enable Always Encrypted → **Enable secure enclaves** → choose protocol.
- Queries must use **parameters** (e.g. `WHERE Salary BETWEEN @Min AND @Max`).
- In-place encryption example:

```sql
ALTER TABLE dbo.Employees
ALTER COLUMN SSN CHAR(11) COLLATE Latin1_General_BIN2
ENCRYPTED WITH (COLUMN_ENCRYPTION_KEY = CEK1, ENCRYPTION_TYPE = RANDOMIZED,
  ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256') NOT NULL
WITH (ONLINE = ON);
```

### Limits to know
- No clustered index, primary key, foreign key or unique constraint on enclave-enabled **randomized** columns.
- Not for memory-optimized tables.
- Turn on **Accelerated Database Recovery (ADR)** before indexing randomized enclave columns. ADR is on by default in Azure SQL, off by default in SQL Server.
- Every **replica** (availability group, geo-replica) must support enclaves, or enclave queries fail after failover.

## Encrypted connections (TLS)
**TLS** (Transport Layer Security) encrypts data **while it moves** between the app and the database. Without it, someone on the network could read passwords and query results.

### Azure SQL Database and Managed Instance
- TLS is used **by default**.
- Set **Minimum TLS version** (server → **Networking → Connectivity**, or `az sql server update --set minimalTlsVersion="1.2"`).
- **TLS 1.0 and 1.1 retired on 31 Aug 2025.** The lowest minimum you can set is **1.2**. TLS 1.3 is also accepted.
- Once you enforce a minimum version, you cannot go back to the default.
- In the app: `Encrypt=True;TrustServerCertificate=False;`.

### SQL Server on a VM
1. Install a valid **TLS certificate** on the server. It must be in the local machine (or current user) store, not expired, and readable by the SQL Server service account. If not, SQL Server **may fail to start**.
2. Open **SQL Server Configuration Manager** → Network Configuration → Protocols for your instance → Properties.
3. **Certificate** tab: pick the certificate. **Flags** tab: **Force Encryption = Yes**.
4. SQL Server **2022+** also has **Force Strict Encryption** (TDS 8.0: the whole session is wrapped in TLS; needed for **TLS 1.3**).
5. **Restart** SQL Server.

In **SSMS 20+**, set **Encryption** on the Login tab to **Mandatory** or **Strict**. Tick **Trust server certificate** only if you must.

## Compare at a glance

| | TDE | Always Encrypted | TLS |
|--|-----|------------------|-----|
| Protects | Whole database + backups on disk | Chosen columns | Data on the network |
| Server sees plain data? | Yes | **No** | n/a |
| Hides data from DBA? | No | **Yes** | No |
| App changes? | None | Driver + connection setting | Connection string |
| Default in Azure SQL? | Yes | No | Yes |

## Remember for the exam
- "Stolen backup or disk" → **TDE**. "Hide data from DBAs / cloud admins" → **Always Encrypted**.
- TDE is on by default in Azure SQL; certificate backup is vital on SQL Server; copy the cert to all AG replicas.
- Customer-managed TDE key lives in **Key Vault**; deleting or disabling it makes the database inaccessible; auto-rotation within **24 hours**.
- Always Encrypted: CMK protects CEK; CEK encrypts data. App needs `Column Encryption Setting=Enabled`.
- Deterministic = equality and joins. Randomized = safer, no queries without enclaves.
- Range / LIKE / ORDER BY on encrypted data, or encrypt a big column without moving data → **secure enclaves**, **randomized**, enclave-enabled keys.
- **VBS** = software, any Azure SQL DB tier, attestation **None**, the only type for SQL Server. **SGX** = DC-series only, **MAA** required, retiring **31 Oct 2027**. **HGS** = SQL Server only.
- **No enclaves** on Managed Instance or Linux.
- SQL Server enclave: `sp_configure 'column encryption enclave type', 1` + **restart**.
- TLS minimum is **1.2** in Azure SQL (1.0/1.1 retired 31 Aug 2025).

## Quick check
1. Auditors require that DBAs must never see credit card numbers, but the app must still search by exact card number. Which feature and type?
   <details><summary>Answer</summary>Always Encrypted with deterministic encryption. It hides data from DBAs and still allows equality lookups.</details>
2. You need LIKE searches on an encrypted column in Azure SQL Database at the lowest cost. What do you configure?
   <details><summary>Answer</summary>Always Encrypted with VBS secure enclaves (preferredEnclaveType = VBS), randomized encryption, enclave-enabled keys, and Attestation Protocol = None. VBS works on any tier.</details>
3. A company disables its TDE key in Azure Key Vault. What happens to the database?
   <details><summary>Answer</summary>It becomes inaccessible, because the server can no longer unlock the database encryption key.</details>
