### ✅ **Summary: Performance and Security in SQL Server on Azure Virtual Machines**

---

### ⚙️ **Storage Considerations**

- **Azure Managed Disks** are the primary storage type for SQL Server on Azure VMs.
- **Key disk types** for SQL workloads:

| Disk Type          | Best For                         | Max IOPS | Max Throughput |
| ------------------ | -------------------------------- | -------- | -------------- |
| **Ultra Disk**     | High IO workloads (critical SQL) | 160,000  | 10,000 MB/s    |
| **Premium SSD v2** | Performance-sensitive workloads  | 80,000   | 1,200 MB/s     |
| **Premium SSD**    | General SQL workloads            | 20,000   | 900 MB/s       |
| **Standard SSD**   | Lightweight workloads            | 6,000    | 750 MB/s       |
| **Standard HDD**   | Backups, non-critical workloads  | 2,000    | 500 MB/s       |

- **Disk Roles:**

  - **OS Disk:** C:\ (Windows) — contains the operating system.
  - **Temporary Disk:** D:\ — used for temp files only, lost on reboot.
  - **Data Disks:** For databases and logs. Use pools and caching for optimized performance.

- **Best Practices:**

  - Use **Premium SSDs** with **read caching** for data files.
  - Keep **transaction logs** on separate disks without caching.
  - TempDB can use the VM's temporary disk or its own dedicated disk.

- **Encryption Options:**

  - **Server-side encryption** (at rest, managed by Azure).
  - **Azure Disk Encryption** (inside VM using BitLocker or DM-Crypt).

---

### 🔒 **Security Options**

- **Microsoft Defender for SQL:** Detect vulnerabilities, assess security, and generate alerts.
- **Azure Security Center:** Centralized security monitoring across hybrid environments.

---

### 🚀 **Performance Considerations**

- Most on-premises SQL performance features work the same in Azure VMs.
- Key features to optimize:

#### 1. **Table Partitioning**

- Improves query and maintenance performance.
- Example: Partitioning by month using filegroups.

#### 2. **Data Compression**

- Reduces IO and improves query performance.
- Types:

  - **Row Compression:** Smaller footprint, low CPU overhead.
  - **Page Compression:** More aggressive, uses prefix & dictionary compression.
  - **Columnstore Archival Compression:** For rarely accessed historical data.

#### Example: Partition Creation

```sql
CREATE PARTITION FUNCTION PartitionByMonth (datetime2)
    AS RANGE RIGHT FOR VALUES ('20210101', '20210201', ..., '20211201');

CREATE PARTITION SCHEME PartitionByMonthSch
    AS PARTITION PartitionByMonth TO (FG1, FG2, FG3, ..., FG12);

CREATE TABLE Order ([Id] int PRIMARY KEY, OrderDate datetime2)
    ON PartitionByMonthSch (OrderDate);
```

---

### ⚙️ **Other Recommended Practices**

- Enable **backup compression**
- Use **instant file initialization**
- Move **system databases and logs to data disks**
- Limit **autogrowth**, disable **autoshrink/autoclose**
- Set **max SQL Server memory**
- Enable **lock pages in memory** (for consistent memory usage)
- Enable **Optimize for Adhoc Workloads**
- Enable **Query Store**
- Regularly run **DBCC CHECKDB**, rebuild indexes, and update statistics
- Manage **transaction log sizes**

---
