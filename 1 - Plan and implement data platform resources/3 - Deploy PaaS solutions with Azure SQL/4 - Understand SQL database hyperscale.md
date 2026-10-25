## ✅ **What is Hyperscale in Azure SQL Database?**

Hyperscale is a **highly scalable service tier** in Azure SQL Database that removes many of the size and performance limits of traditional cloud databases.

- **Old Limit:** Before Hyperscale, a single Azure SQL Database could store **up to 4 TB**.
- **With Hyperscale:** Storage grows automatically from 10 GB up to **128 TB** per database (100 TB per Hyperscale elastic pool). 🔄

It's designed for **large databases**, heavy workloads, and businesses needing **rapid growth, fast backups, and quick scaling.**

---

## 🔑 **Key Features of Hyperscale**

### 1. **Independent Scaling of Compute and Storage:**

- You can **scale up or down** CPU and memory **independently** of the data storage.
- Add **more compute replicas** to handle heavy read workloads.

### 2. **Architecture:**

- Compute (query processing) and storage are **separated**.
- Storage grows **automatically as data grows**, without downtime.
- Multiple compute nodes (**replicas**) can be added for scaling **read workloads**.

### 3. **Rapid Backup and Restore:**

- Backups and restores happen **within minutes**, regardless of database size.
- Backups are stored in **Azure Blob storage** using file snapshots.

### 4. **Read Scale-out:**

- You can create **read-only replicas** for reporting or analytics.
- Connection strings can be configured to route reads to these replicas using:

  ```plaintext
  ApplicationIntent=ReadOnly
  ```

- Replica types: up to **4 high-availability (HA) replicas**, up to **30 named replicas** (each with its own compute size), and **geo-replicas**. 🔄

---

## 🔒 **Security Features:**

Hyperscale supports **all security features of Azure SQL Database**, including:

- **Firewall rules:** Control access by IP address or Virtual Network.
- **Authentication:** SQL Authentication and Microsoft Entra ID. (Windows Authentication for Entra principals is **SQL Managed Instance only**, not Azure SQL Database.) 🔄
- **Data Security:**

  - Transparent Data Encryption (at rest)
  - TLS encryption (in transit)
  - Always Encrypted (in use)
  - Row-Level Security (RLS)
  - Dynamic Data Masking
  - Integration with Azure Key Vault for key management

- **Auditing & Threat Detection:** Detects unusual activities like SQL Injection, suspicious logins, and privilege escalation.

---

## 🚀 **Performance Advantages:**

| Capability                | Traditional SQL DB | Hyperscale SQL DB |
| ------------------------- | ------------------ | ----------------- |
| Max Database Size         | 4 TB               | 128 TB 🔄         |
| Backup/Restore Time       | Hours              | Minutes           |
| Scaling compute resources | Slow & limited     | Fast & dynamic    |
| Read replicas             | Limited            | Multiple replicas |
| Transaction performance   | Standard           | Higher throughput |

---

## ⚙️ **How to Deploy Hyperscale in Azure Portal:**

1. Go to **Azure SQL Database** creation.
2. Under **Compute + storage**, click **Configure database**.
3. Select **Service Tier = Hyperscale**.
4. Choose compute size (vCores), storage, and backup settings.
5. Click **Create**.

---

## 🛑 **Important Considerations:**

- **Reverse migration:** a database migrated to Hyperscale can go back to **General Purpose** only **within 45 days** of the migration (to reach Business Critical, go via General Purpose). Otherwise the move is one-way. 🔄
- **Pricing (simplified Dec 2023):** no SQL software license fee, so it's often cheaper than other tiers for high-performance databases. You pay for vCores per replica + allocated storage. **Azure Hybrid Benefit isn't available for new Hyperscale databases.** 🔄

---

### ➤ **Use Cases:**

- Large-scale **transactional systems**.
- Data warehouses or heavy **reporting workloads**.
- Modernizing large on-premises SQL Server databases.
- Applications with unpredictable or growing workloads.

---
