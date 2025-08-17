# Plan and Implement Data Platform Resources

This section of the DP-300 exam focuses on planning, deploying, configuring, and migrating data platform resources in Azure, particularly Azure SQL solutions. It covers Infrastructure as a Service (IaaS) options like SQL Server on Azure Virtual Machines (VMs), Platform as a Service (PaaS) options like Azure SQL Database and Azure SQL Managed Instance, hybrid setups, scaling for performance, data management features like partitioning and compression, and migration strategies. The goal is to ensure you can recommend, deploy, and optimize SQL workloads in Azure while addressing security, availability, and efficiency.

Real-world context: In a large e-commerce company, you might need to migrate an on-premises SQL Server database handling millions of transactions daily to Azure. This involves evaluating whether to use Azure SQL Database for managed scaling (PaaS) or SQL Server on Azure VMs for more control (IaaS), partitioning tables for better query performance on high-volume order data, and implementing an online migration to minimize downtime during peak shopping seasons.

## 1.1 Plan and Deploy Azure SQL Solutions

This subsection emphasizes recommending the right Azure SQL offering, automating deployments, identifying specialized use cases (e.g., Azure Arc or Fabric integration), planning partitioning and sharding, deploying on various platforms, handling hybrid scenarios, and applying patches/updates.

### Recommend a Database Offering Based on Specific Requirements

To select the optimal database solution, evaluate customer needs, features/benefits, scalability, high availability/disaster recovery (HA/DR), and security. This decides between IaaS (e.g., lift-and-shift from on-premises SQL to Azure VMs) or PaaS (e.g., Azure SQL Database for automated management).

- **Evaluate Customer/Client Needs**: Assess workload type (transactional vs. analytical), data volume, existing infrastructure, and migration complexity. For example, if a client has a legacy SQL Server app requiring full OS control, recommend SQL Server on Azure VMs (IaaS). If they want minimal management, suggest Azure SQL Database (PaaS).
  
- **Features/Benefits of Offerings (PaaS vs. IaaS)**: PaaS (Platform as a Service) like Azure SQL Database handles backups, patching, and scaling automatically, reducing admin overhead. IaaS (Infrastructure as a Service) like SQL Server on Azure VMs offers full customization but requires manual OS/DB management. Hybrid options combine both for flexibility.

- **Scalability**: Consider scaling up (more resources per instance) or out (multiple instances). Azure SQL Database supports hyperscale for massive datasets, while VMs allow custom VM sizing.

- **High Availability and Disaster Recovery (HA/DR)**: Evaluate redundancy options like local/zone/geo-redundancy. Design DR solutions with failover groups or active geo-replication to ensure business continuity.

- **Security Aspects**: Review encryption, authentication, and compliance. PaaS includes built-in Microsoft Defender for SQL, while IaaS requires manual setup.

Real-world example: A financial firm with strict compliance needs might choose Azure SQL Managed Instance (PaaS) for near-100% compatibility with on-premises SQL Server, including features like Transparent Data Encryption (TDE) and Always Encrypted, while scaling to handle end-of-month reporting spikes without downtime.

### Choose an Automated Deployment Method

Automate deployments to ensure consistency, speed, and repeatability using tools like Azure Resource Manager (ARM) templates, Bicep, Azure CLI, or the Azure portal.

- **Azure Resource Manager (ARM) Templates**: JSON files defining resources declaratively. Export from the portal (e.g., from a resource group containing a storage account) and deploy via portal, CLI, or PowerShell.

- **Bicep and Azure CLI**: Bicep is a domain-specific language (DSL) for simpler ARM deployments. Use Azure CLI (Command-Line Interface) with Bicep files; run in Azure Cloud Shell if CLI isn't installed locally. Requires write access to resources and Microsoft.Resources/deployments permissions.

Real-world example: In a DevOps pipeline for a healthcare app, use ARM templates to deploy Azure SQL Database instances across dev/test/prod environments. This ensures identical configs, like vCore count and storage, reducing deployment errors during regulatory audits.

### Identify Use Cases for Azure Arc-Enabled SQL Services

Azure Arc extends Azure management to on-premises, multi-cloud, or edge environments, enabling hybrid SQL Server management with enhanced security and governance.

- **Key Features**: Connects SQL Servers outside Azure to the Azure portal for centralized monitoring. Supports Microsoft Defender for Cloud (vulnerability assessments, threat protection alerts) and Microsoft Purview (data governance across on-premises/multi-cloud/SaaS). Uses Azure AD (Active Directory) authentication over insecure username/password. Handles virtualization and persistent storage on customer infrastructure.

- **Use Cases**: Inventory and assess SQL deployments for best practices; govern data in hybrid setups; secure edge devices (e.g., IoT gateways running SQL).

Real-world example: A manufacturing company with SQL Servers in remote factories uses Azure Arc to scan for vulnerabilities via Microsoft Defender, apply unified governance with Purview, and monitor usage patterns without migrating everything to Azure, ensuring compliance with industry regulations like ISO 27001.

### Identify Use Cases for Azure SQL Database in Microsoft Fabric

Microsoft Fabric integrates data analytics with SQL, creating items in a Fabric workspace: replicated data in OneLake (Parquet format for analytics), SQL analytics endpoint, and default semantic model.

- **Use Cases**: Enable data engineering/science on SQL data without ETL (Extract, Transform, Load); support real-time analytics in Fabric workspaces.

Real-world example: A retail chain uses Azure SQL Database in Fabric to replicate sales data to OneLake, allowing data scientists to build ML models on Parquet files via notebooks, while business users query via the SQL analytics endpoint for dashboards, streamlining inventory forecasting.

### Plan for Table Partitioning

Design partitions based on access patterns to improve query performance, manageability, and scalability in Azure Table Storage or SQL tables.

- **Key Considerations**: Select partition/row keys by data access (e.g., composite keys for multi-property queries). Use Index Table pattern for secondary indexing if needed. Support transactions within partitions; choose shared/unique keys for scaling/grouping.

Real-world example: For a global logistics app storing shipment data, partition by date and region (e.g., "2025-08_US-East"). This allows efficient queries for regional reports and transactional ops on same-partition entities, scaling to handle millions of daily entries without hotspots.

### Recommend a Database Sharding Solution

Sharding distributes data across multiple databases for horizontal scaling. Strategies: Lookup (map tenants to shards), Range (sequence by date/time), Hash (even distribution via hashing).

- **Lookup Strategy**: Application routes requests based on a mapping table (e.g., tenant 1 to Shard A).
- **Range Strategy**: Group by ranges (e.g., orders by date in sequential shards).
- **Hash Strategy**: Use hash functions for balanced load.

Real-world example: A SaaS multi-tenant app shards user data by hashing tenant IDs across shards. For a query on tenant 227, the app hashes to Shard C, preventing overload on any single database during user growth from 10K to 1M.

### Deploy Database Offerings on Selected Platforms

Choose IaaS (SQL Server on Azure VMs) for control or PaaS (Azure SQL Database/Managed Instance/Edge) for management. Azure SQL Database models: Single Database or Elastic Pools.

Real-world example: Deploy Azure SQL Edge on IoT devices in a smart city project for local data processing, syncing to Azure SQL Database for central analytics, balancing edge compute with cloud scale.

### Deploy Hybrid SQL Server Solutions

Combine on-premises and Azure for DR, backups, or management. Use Azure Arc for inventory/assessment; VPN gateways for connectivity.

Real-world example: A bank uses hybrid for DR: On-premises SQL replicates to Azure VMs via IPsec VPN, ensuring failover during outages, with Arc assessing configs for security best practices.

### Apply Patches and Updates for Hybrid and IaaS Deployments

Automate patching for SQL VMs via IaaS Agent extension (configure in portal/PowerShell during provisioning). PaaS like Azure SQL Managed Instance is evergreen (auto-updated).

Real-world example: In an enterprise with hybrid SQL, use Azure Update Manager to schedule patches during off-hours, minimizing disruption to 24/7 operations while keeping systems secure against vulnerabilities like CVE exploits.

## 1.2 Configure Resources for Scale and Performance

Focus on scaling Azure SQL offerings, partitioning tables, and compressing data.

### Configure Azure SQL Database for Scale and Performance

Dynamically add resources; use read scale-out (offload reads to replicas) or sharding (split data). Serverless tier for auto-scaling.

Real-world example: An online gaming platform uses read scale-out to handle read-heavy leaderboards, sharding user profiles across databases for millions of concurrent users.

### Configure Azure SQL Managed Instance for Scale and Performance

Create via portal (basics/networking/additional settings); scale vCores with slider. Supports read scale-out/sharding.

Real-world example: A CRM system scales Managed Instance during sales campaigns, adding vCores minimally downtime, using sharding for customer data partitions.

### Configure SQL Server on Azure VMs for Scale and Performance

Select VM families/sizes; use managed disks for storage perf.

Real-world example: A data warehouse on VMs uses E-series (memory-optimized) with premium SSDs for fast ETL jobs processing terabytes daily.

### Configure Table Partitioning

Create partitioned tables via T-SQL (one/multiple filegroups) or SSMS Wizard. Query metadata for boundaries/rows/compression.

Real-world example: Partition a sales table by year in a reporting DB, enabling fast queries on recent data and independent backups for archives.

### Configure Data Compression

Row (individual rows), Page (entire pages), Columnstore Archival (high compression for cold data).

Real-world example: Compress archival logs in a compliance DB with Columnstore Archival, reducing storage costs by 90% while improving I/O for audits.

## 1.3 Plan and Implement a Migration Strategy

Evaluate needs, strategies, and execute migrations.

### Evaluate Requirements for a Migration

Use DMA (Data Migration Assistant) for compatibility; MAP Toolkit for planning; DMS (Database Migration Service) for seamless migrations.

Real-world example: Assess an e-commerce DB with DMA, identifying blocking issues like unsupported features before migrating to Azure.

### Evaluate Offline or Online Migration Strategies

Offline: Full downtime during migration. Online: Sync data, minimal cutover downtime. Tools: Azure SQL Migration Extension (offline), Transactional Replication (online).

Real-world example: For a hospital system, use online replication to migrate without interrupting patient records access.

### Implement an Online Migration Strategy

Set up replication: distribution/publication/subscription; monitor/troubleshoot.

Real-world example: Migrate a finance app using replication, syncing trades in real-time, cutting over in seconds during maintenance.

### Implement an Offline Migration Strategy

Assess/prepare/copy/deploy with downtime.

Real-world example: Offline migrate a small HR DB during weekends, simpler for non-critical systems.

### Implement a Migration to Azure

Create DMS instance/project; run activity.

Real-world example: Migrate on-premises inventory DB to Azure SQL using DMS, assessing with DMA first.

### Implement a Migration Between Azure SQL Services

Similar to to-Azure, using DMS for schema/data transfer.

Real-world example: Move from Azure SQL DB to Managed Instance for better compatibility, using DMS to handle schema differences.

### Implement Azure SQL Managed Instance Database Copy and Move

Select DBs in portal, provide source/dest details, start/complete operation (within 24h).

Real-world example: Copy a test DB in Managed Instance to prod for quick setup, ensuring no data loss.

### Troubleshoot a Migration

Address regressions (CE version changes), missing indexes, parameter sniffing, TVFs.

Real-world example: Post-migration, fix slow queries by adding indexes via DMVs, converting multi-statement TVFs to inline for better perf in an analytics workload.
