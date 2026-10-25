# Azure SQL Deployment and Migration: A Comprehensive Guide

This guide consolidates critical information on planning, deploying, configuring, scaling, and migrating SQL Server databases to Azure, covering **SQL Server on Azure Virtual Machines (IaaS)**, **Azure SQL Database (PaaS)**, and **Azure SQL Managed Instance (PaaS)**. It provides a unified resource for database administrators and IT professionals seeking to modernize their SQL Server workloads with minimal disruption, optimal performance, robust security, and efficiency. This includes insights from the DP-300 exam perspective on recommending solutions, automating deployments, handling hybrid scenarios, applying patches, configuring for scale, and implementing migration strategies.

## Why Choose Azure for SQL Server?

Deploying SQL Server workloads in Azure offers flexibility, scalability, reduced infrastructure management, and integration with advanced services like Azure Arc and Microsoft Fabric. Azure provides three primary deployment models, each tailored to specific needs:

- **SQL Server on Azure Virtual Machines (IaaS):** Offers full control over the operating system and SQL Server, ideal for "lift-and-shift" migrations of on-premises setups with minimal changes.
- **Azure SQL Database (PaaS):** A fully managed service for modern cloud-native applications, requiring minimal administrative overhead, with models like Single Database or Elastic Pools.
- **Azure SQL Managed Instance (PaaS):** Combines near-complete SQL Server compatibility with PaaS benefits, perfect for modernizing legacy applications with minimal code changes.

### Real-World Examples

A **retail company** with a product catalog database powers its website, mobile apps, and analytics tools. By moving to **Azure SQL Managed Instance**, the company migrates its database with no code changes, leveraging Azure’s automated backups and high availability. Alternatively, a **financial firm** with strict compliance requirements keeps its production SQL Server on-premises but uses **Azure VMs for disaster recovery** and **Azure Blob Storage for backups**, managed centrally via **Azure Arc**.

In a large e-commerce company, migrate an on-premises SQL Server database handling millions of transactions daily to Azure, evaluating whether to use Azure SQL Database for managed scaling (PaaS) or SQL Server on Azure VMs for more control (IaaS), partitioning tables for better query performance on high-volume order data, and implementing an online migration to minimize downtime during peak shopping seasons.

A financial firm with strict compliance needs might choose Azure SQL Managed Instance (PaaS) for near-100% compatibility with on-premises SQL Server, including features like Transparent Data Encryption (TDE) and Always Encrypted, while scaling to handle end-of-month reporting spikes without downtime.

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

- **SQL Server on Azure VM (IaaS):** Use for full control, legacy SQL versions (e.g., SQL Server 2016), or advanced features like FileStream, PolyBase, or cross-instance transactions. Ideal for retail companies managing multiple apps and SQL versions across locations. Select for workloads requiring OS-level customization but manual management.
- **Azure SQL Database (PaaS):** Best for new applications with unpredictable workloads, simpler setups, or large storage needs. A **bike manufacturing company** might use it for inventory and HR databases to reduce maintenance and leverage auto-scaling. Supports hyperscale for massive datasets and serverless auto-scaling.
- **Azure SQL Managed Instance (PaaS):** Perfect for legacy applications requiring SQL Agent, linked servers, or cross-database queries with minimal code changes. A **sports clothing retailer** could migrate its product catalog database seamlessly. Offers near-100% compatibility with on-premises SQL Server.

To recommend, evaluate workload type (transactional vs. analytical), data volume, existing infrastructure, migration complexity, scalability (up/down/out), HA/DR (local/zone/geo-redundancy), and security (encryption, authentication, compliance).

## Licensing Options for Azure SQL Deployments

Across all deployment models, Azure offers flexible licensing to optimize costs:

1. **Pay-as-you-go:** License included in VM or service cost, ideal for testing or short-term use.
2. **Bring Your Own License (BYOL):** Use existing SQL Server licenses with Software Assurance for cost savings.
3. **Azure Hybrid Benefit:** Apply existing Windows/SQL licenses to reduce costs.
4. **Reserved Instances (IaaS only):** Commit to 1-3 years for discounts.
5. **HA/DR License (IaaS only):** Free licenses for disaster recovery replicas.

## Deployment and Configuration

### Choose an Automated Deployment Method

Automate deployments for consistency using Azure Resource Manager (ARM) templates (JSON files defining resources declaratively, exportable from the portal), Bicep (DSL for simpler ARM), Azure CLI (run in Cloud Shell), or the Azure portal. Requires write access and Microsoft.Resources/deployments permissions.

Real-world example: In a DevOps pipeline for a healthcare app, use ARM templates to deploy Azure SQL Database instances across dev/test/prod environments, ensuring identical configs like vCore count and storage.

The **Azure Marketplace** offers pre-configured templates (e.g., SQL Server 2022 + Windows) to simplify deployment. Use **ARM templates** to automate, avoiding manual configuration.

### VM Types for SQL Workloads (IaaS)

Choosing the right VM size is critical for performance and cost optimization:

| Series       | Best For                                   |
| ------------ | ------------------------------------------ |
| General      | Dev/test, small databases                  |
| Compute-Opt. | High CPU jobs (e.g., app servers)          |
| Memory-Opt.  | Large memory SQL workloads (up to 4TB RAM) |
| Storage-Opt. | High-speed storage use                     |
| GPU          | Machine learning, graphics rendering       |
| HPC/FPGA     | Specialized compute tasks                  |

**Recommendation:** Use **DS3_v2+** for Enterprise workloads and **DS2_v2+** for smaller workloads. Select VM families/sizes; use managed disks for storage perf. Real-world example: A data warehouse on VMs uses E-series (memory-optimized) with premium SSDs for fast ETL jobs processing terabytes daily.

### Storage Configuration (IaaS)

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

### SQL Configuration During Deployment (IaaS)

Configure SQL authentication settings, instance configurations, firewall access, and storage type/performance options during VM setup.

### SQL IaaS Agent Extension

This extension, included with Marketplace VMs, automates patching and backups, integration with **Azure Key Vault** and **Defender for Cloud**, disk and SQL health monitoring, and application of best practices.

### Deploy Database Offerings on Selected Platforms

Choose IaaS for control or PaaS for management. Azure SQL Database models: Single Database or Elastic Pools. Real-world example: Deploy Azure SQL Edge on IoT devices in a smart city project for local data processing, syncing to Azure SQL Database for central analytics.

For Azure SQL Managed Instance, create via portal (basics/networking/additional settings).

### Identify Use Cases for Azure Arc-Enabled SQL Services

Azure Arc extends Azure management to on-premises, multi-cloud, or edge environments, enabling hybrid SQL Server management with enhanced security and governance. Key features: Connects SQL Servers outside Azure for centralized monitoring; supports Microsoft Defender for Cloud (vulnerability assessments, threat protection alerts) and Microsoft Purview (data governance); uses Azure AD authentication.

Use cases: Inventory and assess SQL deployments for best practices; govern data in hybrid setups; secure edge devices (e.g., IoT gateways running SQL). Real-world example: A manufacturing company with SQL Servers in remote factories uses Azure Arc to scan for vulnerabilities, apply unified governance, and monitor usage patterns without full migration.

### Identify Use Cases for Azure SQL Database in Microsoft Fabric

Microsoft Fabric integrates data analytics with SQL, creating items in a Fabric workspace: replicated data in OneLake (Parquet format for analytics), SQL analytics endpoint, and default semantic model. Use cases: Enable data engineering/science on SQL data without ETL; support real-time analytics. Real-world example: A retail chain uses Azure SQL Database in Fabric to replicate sales data to OneLake for ML models and dashboards.

### Apply Patches and Updates for Hybrid and IaaS Deployments

Automate patching for SQL VMs via IaaS Agent extension (configure in portal/PowerShell). PaaS is evergreen (auto-updated). Use Azure Update Manager to schedule patches during off-hours. Real-world example: In an enterprise with hybrid SQL, minimize disruption to 24/7 operations while securing against vulnerabilities.

## Hybrid Deployments: Bridging On-Premises and Azure

Hybrid setups combine on-premises and Azure for gradual adoption, cost-effective backups, or resilient DR. Use Azure Arc for inventory/assessment; VPN gateways for connectivity.

### Common Hybrid Setups

1. **Disaster Recovery (DR):** Maintain primary SQL Server on-premises; use an Azure VM as failover.
2. **Cloud Backups:** Back up to **Azure Blob Storage** for quick restores.
3. **Azure Arc-enabled SQL:** Manage on-premises, Azure, and multi-cloud SQL Servers from a single dashboard, including configuration checks, inventory, and security scans.
4. **Deploy Hybrid SQL Server Solutions:** Combine for DR/backups/management. Real-world example: A bank replicates on-premises SQL to Azure VMs via IPsec VPN for failover, with Arc assessing configs.

### Networking and Security for Hybrid Setups

- Ensure **Active Directory (AD)** and **DNS** compatibility.
- Connectivity options:
  - **Site-to-Site VPN:** Cost-effective but higher latency.
  - **ExpressRoute:** Faster, private connection (more expensive).
  - **Point-to-Site VPN (Managed Instance):** Suitable for testing.
  - **Public Endpoint (Managed Instance):** Less secure, uses port 3342.

## Performance and Security Optimization

### Storage and Performance Tips

- **Partitioning:** Split data (e.g., monthly sales) for faster queries. Design based on access patterns; select partition/row keys (e.g., composite for multi-property queries). Use Index Table pattern for secondary indexing. Create partitioned tables via T-SQL (one/multiple filegroups) or SSMS Wizard. Query metadata for boundaries/rows/compression. Real-world example: Partition a sales table by year for fast queries on recent data and independent backups. For a global logistics app, partition by date and region (e.g., "2025-08_US-East") for efficient queries and transactions.
- **Compression:**
  - **Row:** Low CPU overhead, compresses individual rows.
  - **Page:** Better compression ratio, entire pages.
  - **Columnstore/Archival:** Ideal for historical/cold data, high compression. Real-world example: Compress archival logs with Columnstore Archival, reducing storage costs by 90%.
- **Best Practices:**
  - Enable **backup compression** and **instant file initialization**.
  - Move logs and system databases off the OS disk.
  - Disable **autoshrink** and **autoclose**.
  - Cap SQL memory usage.
  - Enable **Query Store** to track query plans.
  - Run **DBCC CHECKDB** regularly.

### Configure Resources for Scale and Performance

- **Azure SQL Database:** Dynamically add resources; use read scale-out (offload reads to replicas) or sharding (split data). Serverless tier for auto-scaling. Real-world example: An online gaming platform uses read scale-out for leaderboards, sharding user profiles.
- **Azure SQL Managed Instance:** Scale vCores with slider; supports read scale-out/sharding. Real-world example: A CRM system scales during campaigns.
- **SQL Server on Azure VMs:** Select VM families/sizes; managed disks.

### Recommend a Database Sharding Solution

Sharding distributes data for horizontal scaling. Strategies: Lookup (map tenants to shards), Range (sequence by date/time), Hash (even distribution via hashing). Real-world example: A SaaS app shards user data by hashing tenant IDs.

### Security Measures

- **Encryption:**
  - **At-rest encryption:** Managed by Azure.
  - **BitLocker/DM-Crypt (IaaS):** Use Azure Disk Encryption.
  - **Transparent Data Encryption (TDE, PaaS):** Secures Azure SQL Database and Managed Instance.
  - **Always Encrypted:** For sensitive data.
- **Azure Key Vault:** Manage encryption keys for compliance.
- **Advanced Threat Protection (PaaS):** Row-level security and threat detection. PaaS includes built-in Microsoft Defender for SQL; IaaS requires manual setup.

## High Availability and Disaster Recovery (HADR)

### Azure HA Options

| Feature            | Availability Goal |
| ------------------ | ----------------- |
| Availability Zones | ~99.99%           |
| Availability Sets  | ~99.95%           |

- **Zones:** Span multiple data centers.
- **Sets:** Distribute VMs across hardware racks.

Evaluate redundancy like local/zone/geo-redundancy; design with failover groups or active geo-replication.

### SQL HADR Options for IaaS

1. **Always On Availability Groups (AG):** Replicates to up to nine VMs; synchronous/asynchronous. Ideal for database-level failover.
2. **SQL Failover Cluster Instance (FCI):** Protects entire instance with shared storage; for legacy.
3. **Distributed Availability Groups:** Connects AGs across locations for zero-downtime migrations.

### Disaster Recovery Options

| Option                    | Type                 | Benefit                              |
| ------------------------- | -------------------- | ------------------------------------ |
| SQL Backups to Blob       | Database backup      | Low-cost, geo-redundant              |
| Azure Backup for SQL      | Agent-based          | Long-term retention, auto-management |
| Azure Site Recovery (ASR) | VM-level replication | Full server failover                 |
| Always On AG (Async)      | Database replication | Seamless regional failover           |

For Azure SQL Database: 99.99% uptime with zone redundancy; automated backups with point-in-time restore and 10-year retention.

## Migration Strategies

### Overview and Evaluation

Azure SQL Database eliminates OS/hardware management, offering auto-scaling, automated backups, high availability, and features like Hyperscale (separates compute/storage), Serverless Auto-Scale, Elastic Query, Automatic Tuning, Elastic Jobs, and Query Insights.

Azure SQL Managed Instance supports SQL Agent, linked servers, CLR.

Evaluate requirements using DMA for compatibility/deprecated features; MAP Toolkit for planning; DMS for migrations. Assess workload, data volume, blocking issues like unsupported features.

Choose offline (full downtime) or online (minimal cutover). Real-world example: For a hospital, use online to avoid interrupting access.

### Migration Process

1. **Assess:** Use DMA to identify issues.
2. **Create:** Set up target instance.
3. **Choose Mode:** Online (no downtime, e.g., Transactional Replication) or Offline (downtime, e.g., DMA/BACPAC).
4. **Migrate:** Transfer schema/data.
5. **Optimize:** Fine-tune.

### Migration Tools

| Tool / Method                 | Type    | Use Case                                     |
| ----------------------------- | ------- | -------------------------------------------- |
| Azure DMS                     | Offline | Large, complex migrations                    |
| Transactional Replication     | Online  | Real-time sync for live systems              |
| Azure Migrate                 | Offline | Full VM or database migration                |
| Import/Export Wizard / BACPAC | Offline | One-time migrations; limitations (>200 GB, excludes logins/jobs) |
| Bulk Copy (`bcp`)             | Offline | Table-level data transfer                    |
| Azure Data Factory            | Offline | Scheduled, filtered migrations               |
| DMA                           | Offline | Schema/data for small/medium DBs; compatibility check |
| Log Replay Service (LRS)      | Online  | Controlled sync with log backups; modes: Autocomplete/Continuous |
| Migration Extension (ADS)     | Online  | Uses DMS for small/medium                    |
| Managed Instance Link         | Online  | Live sync for hybrid; one-way (2016/2019), two-way (2022 preview) |
| Native Backup/Restore         | Offline | Restore `.bak` to Azure                      |
| Log Shipping                  | Online  | Ongoing sync with small downtime             |
| Always On AG                  | Online  | Zero-downtime                                |
| Detach/Attach                 | Offline | Manual large DB moves                        |
| Disk Import Service           | Offline | Physical drive shipping                      |
| Azure SQL Migration Extension | Both    | Online/offline using DMS/backups             |

**Tip:** Use DMS for large/complex; Transactional Replication for 24/7 systems. For partial data: Keep sensitive on-premises; use `bcp` or Data Factory.

**BACPAC Files:** Compressed schema/data archive. Use `SqlPackage.exe /a:import /sf:AdventureWorks.bacpac`. Requires downtime.

**Transactional Replication:** Snapshot initial data, sync via logs. Components: Publisher/Subscriber/Distributor/Article/Publication/Subscription. Limitations: Only transactional; limited schema changes.

**Log Replay Service (LRS):** Upload backups to Blob; Azure syncs. Security: SAS tokens/Managed Identity. Must complete in 30 days.

**Managed Instance Link:** Real-time syncing. Example: Sync catalog to Azure for gradual transition.

**Backup and Restore with Blob Storage:** `BACKUP LOG mydb TO URL = 'https://...'`. Use AzCopy; organize in folders.

**DMA Features:** Migrates schema/data/logins (excludes certificates/`sa`). Best practices: Avoid production; use shared folders; encrypt traffic; run outside hours.

For migration between Azure SQL services: Use DMS for schema/data transfer. Example: From SQL DB to Managed Instance.

Implement Azure SQL Managed Instance Database Copy and Move: Select DBs in portal, provide details, complete within 24h. Example: Copy test to prod.

### Implement an Online Migration Strategy

Set up replication: distribution/publication/subscription; monitor/troubleshoot. Example: Migrate finance app with real-time sync, seconds cutover.

### Implement an Offline Migration Strategy

Assess/prepare/copy/deploy with downtime. Example: Migrate small HR DB during weekends.

### Implement a Migration to Azure

Create DMS instance/project; run activity. Example: Migrate inventory DB, assessing with DMA.

### Performance Tips for Migration

Address I/O bottlenecks; use high-tier during migration then scale down; disable auto-statistics temporarily; place files in target region; implement exponential backoff retries (e.g., 5s → 10s).

**Retry Logic Example:** If SELECT fails: Wait 5s, retry with new connection, increase if needed.

### Troubleshoot a Migration

Address regressions (CE version changes), missing indexes, parameter sniffing, TVFs (convert multi-statement to inline). Example: Fix slow queries by adding indexes via DMVs.

## Monitoring and Automation

- **Azure Data Studio Migration Dashboard:** Tracks progress.
- **Azure Portal:** Monitors DMS.
- **PowerShell/Azure CLI:** Automates. Example: `New-AzDataMigrationToSqlDb -TargetDbName AdventureWorks`.

## Key Takeaways

- **SQL Server on Azure VM:** Maximum control for lift-and-shift, legacy systems.
- **Azure SQL Database:** Simplifies for new apps with elastic scaling, Hyperscale.
- **Azure SQL Managed Instance:** Balances compatibility and PaaS for existing apps.
- **Migration Strategy:** Tools like DMA/DMS/LRS based on downtime/size/sensitivity.
- **Performance and Security:** Optimize storage/partitioning/compression; enable encryption/Query Store.
- **Hybrid Flexibility:** Azure Arc/Blob/Managed Instance Link for integration.
- **Scale and Automation:** Use ARM/Bicep for deployments; sharding/scale-out for performance.

By planning with tools like DMA and leveraging Azure’s best practices, achieve smooth transitions with enhanced performance, security, and scalability.
