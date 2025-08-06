# Azure SQL Deployment and Migration: A Comprehensive Guide

This guide consolidates critical information on deploying and migrating SQL Server databases to Azure, covering **SQL Server on Azure Virtual Machines (IaaS)**, **Azure SQL Database (PaaS)**, and **Azure SQL Managed Instance (PaaS)**. It provides a unified resource for database administrators and IT professionals seeking to modernize their SQL Server workloads with minimal disruption, optimal performance, and robust security.

## Why Choose Azure for SQL Server?

Deploying SQL Server workloads in Azure offers flexibility, scalability, and reduced infrastructure management. Azure provides three primary deployment models, each tailored to specific needs:

- **SQL Server on Azure Virtual Machines (IaaS):** Offers full control over the operating system and SQL Server, ideal for "lift-and-shift" migrations of on-premises setups with minimal changes.
- **Azure SQL Database (PaaS):** A fully managed service for modern cloud-native applications, requiring minimal administrative overhead.
- **Azure SQL Managed Instance (PaaS):** Combines near-complete SQL Server compatibility with PaaS benefits, perfect for modernizing legacy applications with minimal code changes.

### Real-World Example

A **retail company** with a product catalog database powers its website, mobile apps, and analytics tools. By moving to **Azure SQL Managed Instance**, the company migrates its database with no code changes, leveraging Azure’s automated backups and high availability. Alternatively, a **financial firm** with strict compliance requirements keeps its production SQL Server on-premises but uses **Azure VMs for disaster recovery** and **Azure Blob Storage for backups**, managed centrally via **Azure Arc**.

## Comparing Azure SQL Deployment Options

| Feature                       | Azure SQL Database | Azure SQL Managed Instance | SQL Server on Azure VM |
| ----------------------------- | ------------------ | -------------------------- | ---------------------- |
| **Code Changes Needed**       | Many               | Few or none                | None                   |
| **Advanced SQL Support**      | Limited            | Full                       | Full                   |
| **OS Access**                 | No                 | No                         | Yes                    |
| **Admin Effort**              | Low                | Lower                      | High                   |
| **Ideal For**                 | New cloud apps     | Modernizing existing apps  | Re-hosting as-is       |
| **SQL Config Customization**  | Partial            | Extensive                  | Full                   |
| **Network & Storage Control** | No                 | Partial (subnet-based)     | Full                   |

### When to Choose Each Option

- **SQL Server on Azure VM (IaaS):** Use for full control, legacy SQL versions (e.g., SQL Server 2016), or advanced features like FileStream, PolyBase, or cross-instance transactions. Ideal for retail companies managing multiple apps and SQL versions across locations.
- **Azure SQL Database (PaaS):** Best for new applications with unpredictable workloads, simpler setups, or large storage needs. A **bike manufacturing company** might use it for inventory and HR databases to reduce maintenance and leverage auto-scaling.
- **Azure SQL Managed Instance (PaaS):** Perfect for legacy applications requiring SQL Agent, linked servers, or cross-database queries with minimal code changes. A **sports clothing retailer** could migrate its product catalog database seamlessly.

## Licensing Options for Azure SQL Deployments

Across all deployment models, Azure offers flexible licensing to optimize costs:

1. **Pay-as-you-go:** License included in VM or service cost, ideal for testing or short-term use.
2. **Bring Your Own License (BYOL):** Use existing SQL Server licenses with Software Assurance for cost savings.
3. **Azure Hybrid Benefit:** Apply existing Windows/SQL licenses to reduce costs.
4. **Reserved Instances (IaaS only):** Commit to 1-3 years for discounts.
5. **HA/DR License (IaaS only):** Free licenses for disaster recovery replicas.

## Deployment and Configuration for SQL Server on Azure VMs

### VM Types for SQL Workloads

Choosing the right VM size is critical for performance and cost optimization:

| Series       | Best For                                   |
| ------------ | ------------------------------------------ |
| General      | Dev/test, small databases                  |
| Compute-Opt. | High CPU jobs (e.g., app servers)          |
| Memory-Opt.  | Large memory SQL workloads (up to 4TB RAM) |
| Storage-Opt. | High-speed storage use                     |
| GPU          | Machine learning, graphics rendering       |
| HPC/FPGA     | Specialized compute tasks                  |

**Recommendation:** Use **DS3_v2+** for Enterprise workloads and **DS2_v2+** for smaller workloads. The **Azure Marketplace** offers pre-configured templates (e.g., SQL Server 2022 + Windows) to simplify deployment.

### Storage Configuration

Storage performance is critical for SQL Server:

| Disk Type      | Use Case               | Max IOPS | Max Speed   |
| -------------- | ---------------------- | -------- | ----------- |
| Ultra Disk     | Critical SQL workloads | 160K     | 10,000 MB/s |
| Premium SSD v2 | High-performance DBs   | 80K      | 1,200 MB/s  |
| Premium SSD    | Most SQL DBs           | 20K      | 900 MB/s    |
| Standard SSD   | Small/light workloads  | 6K       | 750 MB/s    |
| Standard HDD   | Backups                | 2K       | 500 MB/s    |

**Disk Setup:**

- **OS Disk:** Hosts Windows system files.
- **Temp Disk (D:):** For temporary files only.
- **Data Disks:** Store data and logs with read/write caching as needed.

### SQL Configuration During Deployment

Configure the following during VM setup:

- SQL authentication settings
- Instance configurations
- Firewall access
- Storage type and performance options

**Best Practice:** Use **ARM templates** to automate deployments, avoiding manual configuration in the Azure Portal.

### SQL IaaS Agent Extension

This extension, included with Marketplace VMs, automates:

- Patching and backups
- Integration with **Azure Key Vault** and **Defender for Cloud**
- Disk and SQL health monitoring
- Application of best practices

## Hybrid Deployments: Bridging On-Premises and Azure

Hybrid setups combine on-premises and Azure environments for gradual cloud adoption, cost-effective backups, or resilient disaster recovery.

### Common Hybrid Setups

1. **Disaster Recovery (DR):**
   - Maintain primary SQL Server on-premises.
   - Use an Azure VM as a failover during disasters.
2. **Cloud Backups:**
   - Back up to **Azure Blob Storage** for quick restores during failures.
3. **Azure Arc-enabled SQL:**
   - Manage on-premises, Azure, and multi-cloud SQL Servers from a single Azure dashboard, including configuration checks, inventory, and security scans.

### Networking and Security for Hybrid Setups

- Ensure **Active Directory (AD)** and **DNS** compatibility between on-premises and Azure.
- Connectivity options:
  - **Site-to-Site VPN:** Cost-effective but higher latency.
  - **ExpressRoute:** Faster, private connection (more expensive).
  - **Point-to-Site VPN (Managed Instance):** Suitable for testing from a single machine.
  - **Public Endpoint (Managed Instance):** Less secure, uses port 3342.

## Performance and Security Optimization

### Storage and Performance Tips

- **Partitioning:** Split data (e.g., monthly sales) for faster queries.
- **Compression:**
  - **Row:** Low CPU overhead.
  - **Page:** Better compression ratio.
  - **Columnstore:** Ideal for historical data.
- **Best Practices:**
  - Enable **backup compression** and **instant file initialization**.
  - Move logs and system databases off the OS disk.
  - Disable **autoshrink** and **autoclose**.
  - Cap SQL memory usage.
  - Enable **Query Store** to track query plans.
  - Run **DBCC CHECKDB** regularly for database integrity.

### Security Measures

- **Encryption:**
  - **At-rest encryption:** Managed by Azure for all deployment models.
  - **BitLocker/DM-Crypt (IaaS):** Use Azure Disk Encryption inside VMs.
  - **Transparent Data Encryption (TDE, PaaS):** Secures Azure SQL Database and Managed Instance.
- **Azure Key Vault:** Securely manage encryption keys for compliance.
- **Advanced Threat Protection (PaaS):** Enhances security with row-level security and threat detection.

## High Availability and Disaster Recovery (HADR)

### Azure HA Options

| Feature            | Availability Goal |
| ------------------ | ----------------- |
| Availability Zones | ~99.99%           |
| Availability Sets  | ~99.95%           |

- **Zones:** Span multiple data centers to protect against site failures.
- **Sets:** Distribute VMs across hardware racks to mitigate hardware issues.

### SQL HADR Options for IaaS

1. **Always On Availability Groups (AG):**
   - Replicates databases to up to nine SQL VMs.
   - Supports synchronous (fast) and asynchronous (for remote sites) replication.
   - Ideal for modern applications needing database-level failover.
2. **SQL Failover Cluster Instance (FCI):**
   - Protects the entire SQL instance using shared storage.
   - Best for legacy systems.
3. **Distributed Availability Groups (AG):**
   - Connects two AGs across locations (e.g., on-premises to Azure) for near-zero downtime migrations.

### Disaster Recovery Options

| Option                    | Type                 | Benefit                              |
| ------------------------- | -------------------- | ------------------------------------ |
| SQL Backups to Blob       | Database backup      | Low-cost, geo-redundant              |
| Azure Backup for SQL      | Agent-based          | Long-term retention, auto-management |
| Azure Site Recovery (ASR) | VM-level replication | Full server failover                 |
| Always On AG (Async)      | Database replication | Seamless regional failover           |

## Migration to Azure SQL Database

### Overview

Azure SQL Database is a fully managed PaaS solution that eliminates OS and hardware management, offering auto-scaling, automated backups, and high availability.

### Benefits

- **Backup:** Automated with point-in-time restore and 10-year retention.
- **High Availability:** 99.99% uptime with zone redundancy.
- **Performance:** Supports elastic pools, sharding, and Hyperscale for large databases.
- **Azure-Only Features:**
  - **Hyperscale:** Separates compute and storage for massive databases.
  - **Serverless Auto-Scale:** Adjusts resources dynamically.
  - **Elastic Query:** Queries across databases.
  - **Automatic Tuning:** Manages indexes for performance.
  - **Elastic Jobs:** Schedules tasks across databases.
  - **Query Insights:** Analyzes and optimizes slow queries.

### Migration Process

1. **Assess:** Use **Data Migration Assistant (DMA)** to identify compatibility issues and performance opportunities.
2. **Create:** Set up the Azure SQL Database instance.
3. **Choose Migration Mode:**
   - **Online:** No downtime, using tools like Transactional Replication.
   - **Offline:** Downtime required, using tools like DMA or BACPAC.
4. **Migrate:** Transfer schema and data.
5. **Optimize:** Fine-tune post-migration for performance.

### Migration Tools

| Tool                          | Type    | Use Case                             |
| ----------------------------- | ------- | ------------------------------------ |
| Azure DMS                     | Offline | Large, complex migrations            |
| Transactional Replication     | Online  | Real-time sync for live systems      |
| Azure Migrate                 | Offline | Full VM or database migration        |
| Import/Export Wizard / BACPAC | Offline | One-time migrations                  |
| Bulk Copy (`bcp`)             | Offline | Table-level data transfer            |
| Azure Data Factory            | Offline | Scheduled, filtered migrations       |
| DMA                           | Offline | Schema and data for small/medium DBs |

**Tip:** Use **Azure DMS** for large, complex migrations and **Transactional Replication** for 24/7 critical systems.

### Performance Tips

- Address I/O bottlenecks before migration.
- Use a high-tier database (e.g., Hyperscale) during migration, then scale down.
- Disable auto-statistics updates temporarily.
- Place files in the target region to reduce latency.
- Implement **exponential backoff** for retries (e.g., 5s → 10s → 20s).

**Retry Logic Example:**
If a SELECT query fails:

1. Wait 5 seconds.
2. Retry with a new connection.
3. Increase wait time if needed.

### BACPAC Files

A `.bacpac` file is a compressed archive of schema and data, ideal for offline migrations.

**How to Use:**

```bash
SqlPackage.exe /a:import /sf:AdventureWorks.bacpac
```

**Limitations:**

- Requires downtime.
- Excludes logins, jobs, and linked servers.
- Not suitable for databases >200 GB.

### Transactional Replication

For online migrations with no downtime:

1. Snapshot initial data.
2. Sync changes via SQL Server logs.
3. Push updates to Azure in real-time using **Push subscriptions**.

**Components:**

- **Publisher:** Source SQL Server.
- **Subscriber:** Azure SQL Database.
- **Distributor:** Coordinates replication.
- **Article:** Table/view being replicated.
- **Publication:** Group of articles.
- **Subscription:** Data request from Azure SQL.

**Limitations:**

- Supports only Transactional Replication (no merge or peer-to-peer).
- Limited schema changes during replication.
- SQL Agent jobs require manual management.

### Partial Data Migration

**Why:** Keep sensitive data on-premises, migrate only active data, or reduce cloud storage costs.

**Example:** A bike company migrates only **products and customers** to Azure SQL Database, retaining **sales data** on-premises.

**Tools:**

- **bcp Utility:** Fast, scriptable table-level transfers (schema must exist).
  ```bash
  bcp AdventureWorks.dbo.Product out product_data.txt
  ```
- **Azure Data Factory:** Graphical interface for scheduled, filtered migrations.

## Migration to Azure SQL Managed Instance

### Overview

Azure SQL Managed Instance offers near-complete SQL Server compatibility with PaaS benefits, supporting advanced features like SQL Agent, linked servers, and CLR.

### Migration Tools

| Tool / Method             | Type    | Use Case                                     |
| ------------------------- | ------- | -------------------------------------------- |
| Log Replay Service (LRS)  | Online  | Controlled sync with transaction log backups |
| Migration Extension (ADS) | Online  | Uses Azure DMS for small/medium databases    |
| Managed Instance Link     | Online  | Live sync for hybrid or gradual migrations   |
| Native Backup/Restore     | Offline | Restore `.bak` files to Azure                |
| Transactional Replication | Both    | Real-time table-level syncing                |

### Log Replay Service (LRS)

**Best For:** Controlled migrations with minimal downtime.
**Process:**

1. Create full and log backups.
2. Upload to Azure Blob Storage.
3. Azure restores and syncs until cutover.
   **Modes:**

- **Autocomplete:** Planned downtime after all backups are uploaded.
- **Continuous:** Live sync with minimal downtime at cutover.
  **Security:** Uses SAS tokens or Managed Identity for Blob access.
  **Limitation:** Migration must complete within 30 days.

### Managed Instance Link

Enables real-time syncing for hybrid setups or disaster recovery:

- **One-way (SQL Server 2016/2019):** On-premises to Azure.
- **Two-way (SQL Server 2022, preview):** Bidirectional sync.
  **Example:** A sports retailer syncs its catalog database to Azure, allowing the website to use Azure while mobile apps transition later.

### Connectivity

Managed Instance resides in a dedicated subnet with encrypted traffic:

- **Point-to-Site VPN:** For testing.
- **Site-to-Site VPN:** Network-wide connection.
- **ExpressRoute:** High-speed, private connection.
- **Public Endpoint:** Less secure, uses port 3342.

## Migration to SQL Server on Azure VMs

### Migration Tools

| Tool/Method         | Use Case                                   |
| ------------------- | ------------------------------------------ |
| Azure Migrate       | Full VM migration                          |
| DMA                 | Schema/data migration, compatibility check |
| Backup & Restore    | Simple offline migration                   |
| Log Shipping        | Ongoing sync with small downtime           |
| Always On AG        | Zero-downtime migration                    |
| Detach/Attach       | Manual large DB moves                      |
| bcp / Import Export | Table-level migrations                     |
| Azure Data Factory  | Multi-source data migration                |
| Disk Import Service | Physical drive shipping for slow networks  |

**Example:** A retail company migrates a 1 TB CRM database using **Always On AG** for zero downtime, syncing data from an on-premises server in Mumbai to Azure.

### Backup and Restore with Blob Storage

Back up directly to Azure Blob Storage:

```sql
BACKUP LOG mydb TO URL = 'https://...'
RESTORE DATABASE mydb FROM URL = 'https://...' WITH NORECOVERY
```

**Tip:** Use **AzCopy** for fast transfers and organize backups in `/FULL`, `/DIFF`, and `/LOG` folders.

### Azure SQL Migration Extension

This Azure Data Studio plugin supports online and offline migrations using Azure DMS:

- Uses existing backups from network shares or Blob Storage.
- Requires **Integration Runtime** for network share access.

### Data Migration Assistant (DMA)

**Features:**

- Checks for compatibility issues and deprecated features.
- Migrates schema, data, and logins (excludes certificate-based logins and `sa` account).
  **Best Practices:**
- Avoid running on production servers.
- Use shared folders for temporary files.
- Enable encryption for migration traffic.
- Run assessments outside business hours.

## Monitoring and Automation

- **Azure Data Studio Migration Dashboard:** Tracks migration progress.
- **Azure Portal:** Monitors DMS progress.
- **PowerShell/Azure CLI:** Automates bulk migrations.
  ```powershell
  New-AzDataMigrationToSqlDb -TargetDbName AdventureWorks
  ```

## Key Takeaways

- **SQL Server on Azure VM:** Offers maximum control and compatibility for lift-and-shift migrations, ideal for legacy systems or advanced features.
- **Azure SQL Database:** Simplifies management for new apps with elastic scaling and Azure-only features like Hyperscale.
- **Azure SQL Managed Instance:** Balances compatibility and PaaS convenience for modernizing existing applications.
- **Migration Strategy:** Choose tools (DMA, DMS, LRS, etc.) based on downtime tolerance, database size, and data sensitivity.
- **Performance and Security:** Optimize storage, enable encryption, and follow best practices like disabling autoshrink and enabling Query Store.
- **Hybrid Flexibility:** Use Azure Arc, Blob Storage, and Managed Instance Link for seamless on-premises and cloud integration.

By carefully planning your migration and leveraging Azure’s tools and best practices, you can achieve a smooth transition to the cloud with enhanced performance, security, and scalability.
