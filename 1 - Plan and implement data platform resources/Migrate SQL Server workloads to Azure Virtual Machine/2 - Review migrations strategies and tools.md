## ✅ **Why Careful Migration Planning Is Important**

When migrating SQL Server to Azure VMs, you must plan for:

- **Why:** Understand the **benefits** you'll gain after migration (better performance, easy scaling, improved reliability).
- **How:** Choose the **right tools and methods** to move your data.

A key factor is deciding **how much downtime is acceptable**:

- Some businesses want **zero downtime**.
- Others are okay with **minutes or hours** of downtime.

---

## 🛡️ **Benefits of Running SQL Server on Azure VM**

- Full control over the **operating system** and **SQL Server instance**
- Can run advanced SQL features like:

  - SQL transactional replication
  - Always On Availability Groups
  - Integration/Analysis/Reporting Services
  - FileStream, PolyBase, etc.

- Support for **large databases (up to 256 TB)**
- Keep using your **on-premises SQL Server versions and editions**, no compatibility issues.
- Easier high availability, backups, and updates.

---

## 💰 **Licensing Models**

Three main ways to license SQL Server on Azure VM:

| Model                                                | Description                                                 | When to Use                                                      |
| ---------------------------------------------------- | ----------------------------------------------------------- | ---------------------------------------------------------------- |
| Pay-as-you-go (PAYG)                                 | SQL license **included in VM pricing**                      | Temporary/short-term workloads                                   |
| Bring Your Own License (BYOL) / Azure Hybrid Benefit | Use your **existing SQL licenses**, pay only for VM         | For customers with **Enterprise Agreement & Software Assurance** |
| HA/DR License Model                                  | Free SQL license for passive **disaster recovery replicas** | Disaster recovery setups                                         |

---

## 🌐 **Networking Options**

When you deploy a SQL Server VM, you set the **SQL connectivity type**:

- **Public:** Accessible over the internet (configures firewall and authentication for you).
- **Private:** Accessible only from within your virtual network (more secure).
- **Local:** Accessible only from the same VM (for local use).

---

## 🔑 **Key Management**

Azure Key Vault (AKV) helps store and manage your encryption keys securely.

- SQL Server can integrate with AKV for managing **Transparent Data Encryption (TDE)** keys.
- Makes your environment **more secure and compliant**.

---

## ⚙️ **VM Sizing & Performance Optimization**

To ensure good performance:

| Resource         | Recommendation                                                                              |
| ---------------- | ------------------------------------------------------------------------------------------- |
| VM Size          | Enterprise: DS3_v2+, Standard/Web: DS2_v2+                                                  |
| Storage          | Use **Premium SSDs** for production                                                         |
| Disks            | Separate disks for data, log, and TempDB; use at least 2 P30 disks                          |
| Disk Caching     | Enable read caching for data/TempDB disks, no caching for logs                              |
| IOPS             | Use Ultra SSD or disk striping for high I/O workloads                                       |
| I/O & SQL Config | Enable page compression, disable autoshrink, tune autogrowth, move system DBs to data disks |

---

## 🔧 **Migration Tools & Methods**

| Tool / Method                                       | Purpose                                                |
| --------------------------------------------------- | ------------------------------------------------------ |
| Azure SQL Migration extension for Azure Data Studio | Assess and migrate SQL workloads                       |
| Backup & Restore from Azure Blob                    | Simple offline migration                               |
| Detach/Attach from Azure Storage                    | Manually transfer data and log files                   |
| Log Shipping                                        | Keeps a secondary DB up to date using transaction logs |
| Always On Availability Groups                       | Zero-downtime migration                                |
| Azure Migrate                                       | Comprehensive assessment & migration tool              |
| Data Migration Assistant (DMA)                      | Assess compatibility & migrate schema/data             |
| Database Experimentation Assistant (DEA)            | Test workload performance on Azure VMs                 |

---

## 🚀 **Migration Downtime Options**

| Type                     | Downtime   | Method                                                   |
| ------------------------ | ---------- | -------------------------------------------------------- |
| Zero Downtime            | None       | Always On Availability Groups                            |
| Small Maintenance Window | Minutes    | Azure Database Migration Service                         |
| Large Maintenance Window | Hours/Days | Backup & restore, Detach/Attach, or manual import/export |

---

## ✔️ **Summary of Key Steps**

1. Assess your current SQL Servers & workloads.
2. Choose the best migration tool and downtime strategy.
3. Right-size your Azure VM (CPU, RAM, disk, performance).
4. Set up networking and security (SQL connectivity type, AKV, etc.).
5. Perform the migration.
6. Run **post-migration checks** to verify performance.

---
