## 🔐 SQL Server & Azure SQL: Encryption & Security Essentials (TL;DR Edition)

### ✅ **Transparent Data Encryption (TDE)**

Encrypts the _entire database at rest_ (data files, backups, logs) — no app changes needed.

- **Encrypts on disk**, **decrypts in memory**
- Uses AES-256 or 3DES
- Enabled by default in **Azure SQL** post-2017/2019 (SQL DB/MI)
- Supports **BYOK** with Azure Key Vault for key control
- On-prem: Set up with `CREATE MASTER KEY`, certificate, and `ALTER DATABASE ... SET ENCRYPTION ON;`
- If using Always On, copy certs to all replicas
- Use with **Azure Disk Encryption** for VMs for extra protection

---

### ✅ **Always Encrypted**

Encrypts **specific columns** so even DBAs/cloud admins can’t see plaintext.

- Encryption happens **on the client-side**
- SQL Server stores encrypted blobs only
- Uses **Column Master Key (CMK)** and **Column Encryption Key (CEK)**
- Types:

  - **Deterministic** (allows joins/filters, less secure)
  - **Randomized** (more secure, limited querying)

- With **Secure Enclaves**, even randomized columns can be filtered/indexed
- Needs compatible drivers + `Column Encryption Setting=Enabled`

---

### ✅ **Encrypted Connections**

Protects data _in transit_ via **TLS**.

- SQL Server needs a valid cert
- Configure via **SQL Server Configuration Manager**
- Azure SQL uses **TLS by default**
- App must have `Encrypt=True;TrustServerCertificate=False` in connection string

---

### ✅ **Firewalls in Azure SQL**

Controls **who can connect**, based on IP or VNet.

- **Server-level**: Affects all DBs (set via Azure Portal or T-SQL in `master`)
- **Database-level**: Only affects a single DB
- Uses rule-check order: DB-level → Server-level → Block
- Can also use:

  - **VNet Endpoints** (for limited Azure regions)
  - **Private Link** (securest, uses private IP, no public internet access)

🔒 **Best Practices:**

| Use Case                 | Firewall Option               |
| ------------------------ | ----------------------------- |
| Whole company access     | Server-level                  |
| Partner access to one DB | Database-level                |
| App in Azure VNet        | VNet Endpoint or Private Link |
| Top-tier security        | Private Link                  |

---

### ✅ **SQL Injection (SQLi)**

Attackers inject malicious SQL via user input.

- Prevent with:

  - **Parameterized queries**
  - **Stored procedures**
  - **Input validation**
  - **ORMs**, least privilege, WAF, Azure ATP

- Don’t rely on filtering keywords — attackers can bypass with encoding or binary payloads

---

### ✅ **Azure Key Vault Integration**

Securely stores **certs, secrets, keys**.

- SQL Server can use it for:

  - **TDE keys**
  - **Always Encrypted CMKs**
  - **Backup encryption**

- Controlled via **RBAC**, not just Azure admin rights
- Supports **auditing, monitoring, rotation**

---

### ✅ **Encryption Quick Compare**

| Feature                   | TDE                       | Always Encrypted           | TLS Connection                 |
| ------------------------- | ------------------------- | -------------------------- | ------------------------------ |
| Scope                     | Full DB & backups         | Specific sensitive columns | In-transit data                |
| SQL Server sees data?     | Yes                       | No                         | N/A                            |
| Transparent to app?       | Yes                       | No (needs driver + config) | Yes (if connection string set) |
| Protects from DBAs/admins | No                        | Yes                        | No                             |
| Use case                  | Compliance, backup safety | High-security column data  | Prevent sniffing on network    |

---

### ✅ Final Word

Use **TDE** for blanket protection, **Always Encrypted** for ultra-sensitive fields, and **TLS** to lock down network traffic.
🔑 Manage your keys with **Azure Key Vault**, and block uninvited guests with **firewalls + Private Link**.
🛡️ Combine these like armor pieces — because security ain’t one-size-fits-all.
