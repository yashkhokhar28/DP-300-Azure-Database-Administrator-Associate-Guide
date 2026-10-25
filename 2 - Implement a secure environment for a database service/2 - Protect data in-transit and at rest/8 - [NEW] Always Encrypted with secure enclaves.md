## 🔐 **What Are Secure Enclaves (and Why the Exam Cares)?**

- The 27 Oct 2026 DP-300 skill is now **"Implement Always Encrypted with secure enclaves"** (previously worded "...with VBS enclaves"). Know **both** enclave types, not just VBS.
- Plain **Always Encrypted** never lets plaintext data or keys into the Database Engine, so the server can only do **equality on deterministic** columns. Everything else (range, LIKE, sorting, initial encryption, key rotation) must happen **client-side**.
- A **secure enclave** is a **protected region of memory inside the Database Engine process**. It is an opaque box: nothing outside (not even a debugger, the DBA or the OS admin) can see data or code inside it.
- The client driver sends the needed **column encryption keys (CEKs) to the enclave over a secure channel**; the engine delegates computations on encrypted columns to the enclave, which decrypts and works on plaintext **only inside the enclave**.

---

## 🚀 **What Enclaves Add Over Plain Always Encrypted**

| Capability                      | Plain Always Encrypted              | With secure enclaves (enclave-enabled columns)                          |
| ------------------------------- | ----------------------------------- | ----------------------------------------------------------------------- |
| Equality (`=`)                  | Deterministic only                  | Deterministic **and randomized**                                        |
| `<`, `>`, `BETWEEN`, `IN`, `LIKE`, `DISTINCT` | ❌                    | ✅ (randomized columns)                                                 |
| Joins                           | Deterministic equality joins only   | ✅ (SQL Server 2019: **nested loop joins only**)                        |
| `ORDER BY`, `GROUP BY`          | ❌                                  | ✅ Azure SQL Database & SQL Server **2022+** (❌ in SQL Server 2019)    |
| Indexes on randomized columns   | ❌                                  | ✅ **Nonclustered** only (sorted by plaintext inside the B-tree)        |
| Initial encryption / key rotation / change encryption type / decrypt | Client-side tool must pull data out | ✅ **In-place encryption** via `ALTER TABLE ... ALTER COLUMN` |

- ⚠️ Rich queries (range, LIKE, sort...) inside the enclave work on **randomized** encryption. Deterministic enclave-enabled columns still only get **equality**.
- **In-place encryption** works with **both** deterministic and randomized, as long as all keys involved are **enclave-enabled**.
- Character columns used in enclave queries need a **`_BIN2` collation** (Azure SQL DB & SQL Server 2022 also accept **UTF-8** collations; SQL Server 2019 needs BIN2).

---

## 🧩 **Enclave Technologies: VBS vs Intel SGX**

| Feature                     | **VBS enclaves** (Virtualization-based Security, aka VSM)              | **Intel SGX enclaves**                                              |
| --------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------- |
| Type                        | **Software-based**, uses the **Windows hypervisor**; no special hardware | **Hardware-based** trusted execution environment                   |
| SQL Server 2019 / 2022 / 2025 (Windows, on-prem or Azure VM) | ✅ The **only** enclave type supported           | ❌ Not supported                                                    |
| Azure SQL Database          | ✅ **Any** purchasing model/tier/hardware (vCore, DTU, serverless, elastic pools) except **DC-series** | ✅ **Only** with **DC-series** hardware (vCore) |
| Protects against            | Attacks **inside the VM/guest OS** (e.g. memory dumps, DBAs)           | Attacks from **both guest OS and host OS**                          |
| Status                      | ✅ Current, **recommended**                                             | ⚠️ **End of support 31 Oct 2027**                                   |

- **Azure SQL Managed Instance** – ❌ secure enclaves are **not** listed as supported (docs apply to "SQL Server 2019+ on Windows" and "Azure SQL Database" only). Plain Always Encrypted works on MI.
- **SQL Server on Linux** – ❌ not supported (feature is "SQL Server 2019 and later **on Windows**").
- **SGX retirement:** after **31 Oct 2027**, Azure automatically moves databases still on **DC-series** to a standard-series (non-DC) tier and **enables VBS enclaves**. Migration paths: Azure SQL DB with **VBS** enclaves, or **SQL Server on an Azure confidential VM** with VBS (+ optional HGS).
- **DC-series** limits (if you see it in a question): uses **physical** cores, max **40** cores, **no serverless**, limited regions.

---

## 🛡️ **Enclave Attestation**

**Attestation** = the client verifies the enclave is genuine and runs the genuine Microsoft-signed Always Encrypted library **before** sending it keys (defense against a malicious OS admin tampering with the enclave).

| Hosting product                  | Enclave type                     | Supported attestation protocols                     |
| -------------------------------- | -------------------------------- | --------------------------------------------------- |
| SQL Server 2019 and later        | VBS                              | **Host Guardian Service (HGS)** or **None**         |
| Azure SQL Database               | SGX (DC-series)                  | **Microsoft Azure Attestation** – **mandatory**     |
| Azure SQL Database               | VBS                              | **None** only – attestation **not supported**       |

- **Microsoft Azure Attestation (MAA):** create an **attestation provider**, configure the Microsoft-provided **policy**, grant the logical server access, share the **attestation URL** with app admins.
- **Host Guardian Service (HGS):** Windows Server role implementing **Windows Defender System Guard runtime attestation**; SQL Server host is registered with HGS.
- **No attestation** = quickest setup, but a client can't detect a tampered enclave. Requires newer drivers.
- **Role separation:** attestation admin ≠ DBA ≠ app admin. DBAs should **not** control attestation policies.

---

## 🔑 **Enclave-Enabled Keys and Columns**

- **Enclave-enabled CMK** – CMK metadata with the **`ENCLAVE_COMPUTATIONS`** property plus a **signature** (generated client-side by SSMS/PowerShell; SSMS checkbox **"Allow enclave computations"**).
- **Enclave-enabled CEK** – any CEK encrypted with an enclave-enabled CMK. **Only these** can be used inside the enclave.
- **Enclave-enabled column** – a column encrypted with an enclave-enabled CEK.
- Enclave-enabled CMKs can only live in **Windows Certificate Store** or **Azure Key Vault** (no HSM CSP/CNG providers).

```sql
-- Metadata for an enclave-enabled CMK in Azure Key Vault (signature produced by SSMS/PowerShell)
CREATE COLUMN MASTER KEY CMK_Enclave
WITH (
    KEY_STORE_PROVIDER_NAME = N'AZURE_KEY_VAULT',
    KEY_PATH = N'https://contoso-kv.vault.azure.net/keys/AE-CMK/4c05f1a41b12488f9cba2ea964b6a700',
    ENCLAVE_COMPUTATIONS (SIGNATURE = 0xA80F5B12...)
);
```

---

## ⚙️ **How to Enable the Enclave**

**Azure SQL Database – VBS** (off by default; sets **`preferredEnclaveType = VBS`** on the database or elastic pool):

- **Portal:** create DB/pool → **Security** tab → **Always Encrypted** → **Enable secure enclaves = ON**. Existing DB: **Security → Data Encryption → Always Encrypted** tab. Existing pool: **Settings → Configuration → Always Encrypted** tab.
- **SSMS:** New Database / Properties → **Configure SLO** page → **Enable Secure Enclaves = ON**.
- **PowerShell / CLI:**

```powershell
Set-AzSqlDatabase -ResourceGroupName "rg-sql" -ServerName "sql-contoso" `
    -DatabaseName "HRDB" -PreferredEnclaveType VBS
# az sql db update -g rg-sql -s sql-contoso -n HRDB --preferred-enclave-type VBS
```

- Databases added to a VBS-enabled **elastic pool inherit** VBS. Adding a VBS-enabled DB to a pool **without** VBS is **not supported**.

**Azure SQL Database – SGX:** pick **DC-series** hardware (vCore) + configure **Microsoft Azure Attestation**.

**SQL Server 2019+ (on-prem or Azure VM) – VBS:** host needs **Windows Server 2019+** (or Windows 11 24H2+), CPU virtualization (Intel VT-x/EPT or AMD-V/RVI), **VBS running** (`msinfo32`). In Azure use a **Generation 2 VM** (or Gen1 with nested virtualization).

```sql
EXEC sp_configure 'column encryption enclave type', 1;   -- 0 = none, 1 = VBS
RECONFIGURE;
-- ⚠️ Restart the SQL Server instance for it to take effect

SELECT [value], [value_in_use] FROM sys.configurations
WHERE [name] = 'column encryption enclave type';
SELECT * FROM sys.dm_column_encryption_enclave;   -- exactly one row = enclave initialized
```

---

## 🔁 **In-Place Encryption Examples (T-SQL)**

Run from a connection with **Always Encrypted + enclave computations enabled** (SSMS/app). Not supported from `sqlcmd` or `Invoke-Sqlcmd`.

```sql
-- Encrypt a plaintext column in place (switching to a BIN2 collation in the same statement is allowed)
ALTER TABLE dbo.Employees
ALTER COLUMN SSN CHAR(11) COLLATE Latin1_General_BIN2
ENCRYPTED WITH (COLUMN_ENCRYPTION_KEY = CEK1, ENCRYPTION_TYPE = RANDOMIZED,
                ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256') NOT NULL
WITH (ONLINE = ON);
GO
ALTER DATABASE SCOPED CONFIGURATION CLEAR PROCEDURE_CACHE;   -- refresh parameter encryption metadata

-- Rotate the CEK in place: same statement, new key
-- ... ENCRYPTED WITH (COLUMN_ENCRYPTION_KEY = CEK2, ENCRYPTION_TYPE = RANDOMIZED, ...)

-- Confidential queries once enabled (parameters required)
DECLARE @MinSalary MONEY = 40000, @MaxSalary MONEY = 45000;
SELECT * FROM HR.Employees WHERE Salary BETWEEN @MinSalary AND @MaxSalary;
DECLARE @Pattern CHAR(11) = '%6789';
SELECT * FROM HR.Employees WHERE SSN LIKE @Pattern;
```

- ⚠️ You **can't combine** an encrypt/re-encrypt/decrypt with other column changes (e.g. data type) in one `ALTER COLUMN` – only **collation (same code page)** and **NULL/NOT NULL**.

---

## 💻 **Client Driver & SSMS Settings**

| Setting (Microsoft.Data.SqlClient keyword) | Value                                                                  |
| ------------------------------------------ | ---------------------------------------------------------------------- |
| `Column Encryption Setting`                | `Enabled`                                                              |
| `Attestation Protocol`                     | `HGS` (SQL Server + HGS), `AAS` (Azure SQL DB SGX + MAA), `None` (VBS without attestation). If omitted → **enclaves disabled** |
| `Enclave Attestation Url`                  | HGS or MAA endpoint; **not required** when protocol = `None`           |

```text
Server=tcp:sql-contoso.database.windows.net;Database=HRDB;Authentication=Active Directory Default;
Column Encryption Setting=Enabled;Attestation Protocol=None;
```

- **Drivers:** Microsoft.Data.SqlClient (≥ **4.1** for `None`), ODBC **17.4+** (≥ **18.1** for `None`), JDBC **8.2+** (≥ **12.2** for `None`), .NET Framework provider 4.7.2+.
- **SSMS:** Connect → Options → **Always Encrypted** tab → ✅ **Enable Always Encrypted (column encryption)** → ✅ **Enable secure enclaves** → **Protocol**: *Host Guardian Service* (SQL Server), *Microsoft Azure Attestation* (Azure SQL DB SGX) or *None* (Azure SQL DB VBS) → attestation URL (if any) → accept **Parameterization for Always Encrypted**.

---

## ⚠️ **Limitations & Operational Gotchas**

- **No clustered index**, **primary key**, **foreign key** or **unique** constraint on enclave-enabled **randomized** columns.
- Not supported on **in-memory (memory-optimized) tables**; computed columns can't compute on enclave-enabled randomized columns; no escape characters in `LIKE`.
- **Enable Accelerated Database Recovery (ADR)** **before** creating indexes on randomized enclave-enabled columns (ADR is on by default in Azure SQL DB/MI, **off** by default in SQL Server).
- **HA/DR:** every replica (AG secondaries, geo-replicas) must support enclaves, or enclave queries fail after failover.
- Restore to an instance without an enclave → restore succeeds but enclave queries fail and enclave indexes become invalid. Restoring a VBS-enabled DB → **reconfigure the VBS setting**.
- **BACPAC:** drop indexes on randomized enclave-enabled columns before export.
- **Ordering leak:** range-capable columns can reveal order (e.g. the max salary via index scan) – a known trade-off.

---

## ✅ **Summary**

| Question                               | Answer                                                               |
| -------------------------------------- | -------------------------------------------------------------------- |
| What do enclaves add?                  | Rich queries on **randomized** columns + **in-place encryption/rotation** + indexes |
| SQL Server enclave type                | **VBS only** (2019+, Windows); `sp_configure 'column encryption enclave type', 1` + restart |
| Azure SQL DB enclave types             | **VBS** (any tier, no attestation) or **SGX** (DC-series, MAA required, retiring Oct 2027) |
| Managed Instance                       | ❌ No secure enclaves                                                 |
| Key requirement                        | **Enclave-enabled CMK** (`ENCLAVE_COMPUTATIONS`) → enclave-enabled CEK |
| App connection                         | `Column Encryption Setting=Enabled` + `Attestation Protocol` (+ URL)  |

---

## 🎯 **Exam Tips / Traps**

- 🎯 "Run **LIKE / range / ORDER BY** on encrypted SSN/salary without moving data to the client" → **Always Encrypted with secure enclaves** + **randomized** encryption + enclave-enabled keys.
- 🎯 "Encrypt or **rotate keys** on a large column **without exporting data**" → **in-place encryption** (enclave-enabled CEK, `ALTER TABLE ... ALTER COLUMN ... ENCRYPTED WITH`).
- 🎯 "Enable enclaves on Azure SQL Database with **minimum cost / any tier**" → set **`preferredEnclaveType = VBS`** (portal toggle **Enable secure enclaves**). **DC-series** is only for SGX and is being retired.
- 🎯 "Enable enclaves on SQL Server on an Azure VM" → `sp_configure 'column encryption enclave type', 1`, **restart**, verify with `sys.dm_column_encryption_enclave`; optional **HGS** attestation.
- ⚠️ Trap: **Azure SQL DB VBS → Attestation Protocol = None** (no MAA). MAA (`AAS`) is **only** for SGX/DC-series. HGS is **only** for SQL Server.
- ⚠️ Trap: "Enclaves need special hardware (SGX)" – **false** for VBS; only SGX needs DC-series.
- ⚠️ Trap: Enclave rich queries **don't** work on **deterministic** columns (equality only).
- ⚠️ Trap: Managed Instance and SQL Server on Linux → **no** secure enclaves.
- ⚠️ Trap: SQL Server **2019** enclaves: **no ORDER BY / GROUP BY**, only **nested loop** joins.

> Source: https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-enclaves · https://learn.microsoft.com/en-us/azure/azure-sql/database/always-encrypted-enclaves-plan · https://learn.microsoft.com/en-us/azure/azure-sql/database/always-encrypted-enclaves-enable · https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/configure-column-encryption-enclave-type · https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-enclaves-no-attestation-plan · https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-enclaves-query-columns · https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-enclaves-configure-encryption-tsql · https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-enclaves-client-development · https://learn.microsoft.com/en-us/sql/connect/ado-net/sql/sqlclient-support-always-encrypted · https://learn.microsoft.com/en-us/sql/t-sql/statements/create-column-master-key-transact-sql · https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-enclaves-migration
