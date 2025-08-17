# Plan and Configure a High Availability and Disaster Recovery (HA/DR) Environment

This section of the DP-300 exam addresses planning, implementing, and managing High Availability (HA) and Disaster Recovery (DR) strategies for Azure SQL solutions, including SQL Server on Azure Virtual Machines (VMs), Azure SQL Database, and Azure SQL Managed Instance. High Availability ensures minimal downtime during failures, while Disaster Recovery focuses on restoring operations after major disruptions. Key considerations include Recovery Point Objective (RPO - the maximum acceptable data loss measured in time) and Recovery Time Objective (RTO - the maximum acceptable downtime). The focus is on recommending strategies, handling backups/restores, configuring replication technologies, and monitoring/troubleshooting to achieve business continuity.

Real-world context: In a global banking system, where downtime can cost millions per minute, an HA/DR setup might use Always On Availability Groups for zero-downtime failover within a region (HA) and active geo-replication across regions (DR) to meet strict RPO/RTO of 5 seconds/30 seconds, ensuring transaction data is always available even during natural disasters like hurricanes affecting data centers.

## 5.1 Plan an HA/DR Strategy for Database Solutions

This subsection involves assessing requirements and recommending tailored HA/DR approaches, including hybrid and Azure-native options, with testing procedures.

### Recommend HA/DR Strategy Based on Recovery Point Objective/Recovery Time Objective (RPO/RTO) Requirements

Evaluate business needs to select strategies that align with RPO (data loss tolerance) and RTO (downtime tolerance).

- **Options**:
  - **Availability Sets**: Group VMs to avoid single points of failure within a data center.
  - **Availability Zones**: Distribute across physically separate zones in a region for higher resilience.
  - **Azure Site Recovery**: Replicates VMs to another region for DR, supporting automated failover.
  - **Virtual Networks**: Ensure secure connectivity for replicated resources.

Real-world example: An e-commerce platform with RPO of 1 minute and RTO of 5 minutes uses Availability Zones for HA to handle zone outages (e.g., power failure in one zone) and Azure Site Recovery for DR to failover to a secondary region during widespread issues like cyberattacks, minimizing lost orders during Black Friday sales.

### Evaluate HA/DR for Hybrid Deployments

Hybrid setups combine on-premises and Azure, using technologies like Always On Availability Groups, Log Shipping, or Database Mirroring (applicable to SQL Server on VMs).

- **Always On Availability Groups**: Synchronous replication for HA (primary/secondary replicas); supports domain controllers and file share witnesses via VPN tunnels.
- **Log Shipping**: Asynchronous shipping of transaction logs for DR; high-performance but potential data loss.
- **Database Mirroring**: Synchronous/asynchronous mirroring for failover; principal/mirror roles.

Real-world example: A manufacturing company with on-premises SQL Servers uses Always On Availability Groups in hybrid mode, replicating to Azure VMs over a VPN tunnel. This ensures seamless failover during on-site power outages, with a witness server preventing split-brain scenarios, maintaining production line data sync.

### Evaluate Azure-Specific HA/DR Solutions

Azure-native options for pure cloud or VM-based workloads.

- **High Availability (HA)**: Database-level with Always On Availability Groups; instance-level with Failover Cluster Instances (FCI).
- **Disaster Recovery (DR)**: Availability Groups for replication/failover; Azure Site Recovery for VM-level protection; backup/restore to Azure Blob Storage.

Real-world example: A healthcare provider evaluates Azure SQL Database's built-in HA (99.99% uptime via replicas) but opts for Always On Availability Groups on Azure VMs for custom control, adding Azure Site Recovery for DR to replicate to a secondary region, ensuring patient records are recoverable within RTO of 15 minutes after a regional outage.

### Plan a Testing Procedure for an HA/DR Solution

Validate strategies without impacting production.

- **Steps**: Run test failovers on single VMs or recovery plans; configure load balancers for routing; test connectivity post-failover; validate cluster functionality in production-like networks.

Real-world example: An insurance firm plans quarterly DR drills using Azure Site Recovery test failovers on a recovery plan grouping SQL VMs. They simulate a disaster, failover to a test network, verify application connectivity via load balancer, and check data integrity, ensuring compliance with regulations like SOX (Sarbanes-Oxley Act) without real downtime.

## 5.2 Plan and Perform Backup and Restore of a Database

Focus on strategies for data protection, using native tools, T-SQL, and cloud storage for backups/restores, including point-in-time recovery and long-term retention.

### Recommend a Database Backup and Restore Strategy

Choose based on workload, RPO/RTO, and automation needs.

- **Azure Backup for SQL VMs**: Full VM backups including disks via VSS (Volume Shadow Copy Service); restore to point-in-time; store in Recovery Services Vault.
- **Manual Backup**: Schedule via tools for specific databases; backup to attached disks or network shares.
- **Automated Backup**: Regular full/differential/log backups; ensures low RPO with transaction logs.
- **Backup/Restore to Blob Storage**: Direct to Azure Blob for cost-effective, geo-redundant storage.

Real-world example: A media streaming service recommends automated backups for Azure SQL Database to meet RPO of 5 minutes, using Azure Backup for VMs in hybrid setups. For restores, they plan point-in-time to recover from accidental deletions, like a content catalog error during updates.

### Perform a Database Backup by Using Native Tools

Use built-in Azure tools for reliable backups.

- **Azure Data Studio**: Graphical interface to backup/restore; configure dashboards for monitoring.
- **Azure CLI**: Script backups, e.g., create variables for resource group/server/database, then backup to storage container (e.g., `az sql db export` for .bacpac files).

Real-world example: A development team uses Azure Data Studio to backup a test database before schema changes, ensuring quick rollback. For production, they script Azure CLI commands in pipelines to backup Azure SQL Databases to Blob Storage nightly, automating compliance reporting.

### Perform a Database Restore by Using Native Tools

Restore from backups to recover data.

- **Azure Data Studio**: Select restore options for databases.
- **Azure CLI**: Script restores to point-in-time, e.g., create new server/database, use `az sql db restore` with variables for source/target.

Real-world example: After a ransomware attack on a retail DB, use Azure CLI to restore from a clean backup to a new server, minimizing downtime by scripting the process in advance for rapid execution.

### Perform a Database Restore to a Point in Time

Target specific timestamps for recovery.

- **For Azure SQL Managed Instance**: Restore existing/dropped DBs to same/another instance via Azure Portal (UI), Azure CLI (limited), or PowerShell (full support).

Real-world example: In a financial app, restore a transaction DB to 10 minutes before a faulty update using PowerShell on Azure SQL Managed Instance, recovering lost trades without full downtime, meeting audit requirements.

### Configure Long-Term Backup Retention

Extend beyond standard PITR (Point-In-Time Restore) windows.

- **Long-Term Retention (LTR)**: Leverages automated full backups; configure policies to copy to separate blobs; enable PITR for years.

Real-world example: A legal firm configures LTR policies for client case databases, retaining backups for 10 years in Blob Storage, allowing restores for litigation evidence while complying with data retention laws like GDPR.

### Backup and Restore a Database by Using T-SQL

Script-based operations for automation.

- **Backup Options**: COPY_ONLY (non-interfering copy), COMPRESSION/NO_COMPRESSION, ENCRYPTION. Azure SQL Managed Instance supports auto-backups plus user COPY_ONLY.
- **Restore**: Use RESTORE DATABASE with options like NORECOVERY for multi-step restores.

Real-world example: An analytics platform uses T-SQL scripts in jobs to backup databases with COMPRESSION to reduce storage, restoring with ENCRYPTION during migrations to ensure data security in transit.

### Backup to and Restore from Cloud Storage

Use Azure Blob for scalable, secure storage.

- **Path Format**: https://<AccountName>.blob.core.windows.net/<Container>/<File>.bak
- **Authentication**: Storage Account Key (page blobs) or Shared Access Signature (SAS, block blobs - recommended for cost/security from SQL 2016+).
- **Restore**: In SSMS, select URL as media; in T-SQL, use FROM URL.
- **Commands**: BACKUP LOG/TO URL for logs; RESTORE DATABASE FROM URL WITH NORECOVERY for full backups.

Real-world example: A logistics company backs up shipment DBs to Blob using SAS tokens via T-SQL, restoring during DR drills to verify RPO, saving costs over on-premises tape storage.

## 5.3 Create and Manage Database Tasks in Azure

Implement and manage replication/failover technologies specific to Azure.

### Configure Active Geo-Replication

Asynchronous replication to secondary regions for DR.

- **Methods**: Azure Portal (create geo-replica with primary details); Azure CLI (e.g., `az sql db replica create` for setup, `az sql db replica list-links` for status).

Real-world example: A travel booking site configures active geo-replication for its reservation DB, replicating from East US to West US. During an East Coast storm, they failover manually, maintaining bookings with minimal data loss.

### Configure Always On Availability Groups on SQL Managed Instance and Azure Virtual Machines

Synchronous/asynchronous replication for HA/DR.

- **Process**: Use Azure Portal to set up groups, replicas, listeners.

Real-world example: An enterprise ERP (Enterprise Resource Planning) system on Azure VMs uses Always On for synchronous replicas in the same region (HA) and async to another (DR), ensuring zero data loss during hardware failures.

### Configure Failover Groups

Manage replication and automated failover for databases or Managed Instances.

- **Features**: Read-write listener for primary; read-only for secondaries; auto-failover based on policies.

Real-world example: A SaaS provider uses failover groups for customer DBs, automatically switching to a secondary region during outages, with read-only access for reporting offloading load.

### Configure Always On Failover Cluster Instances on Azure Virtual Machines

Shared-storage clustering for instance-level HA.

- **Components**: Availability Sets, Azure Load Balancer, Windows Server Failover Cluster, Storage Spaces Direct; supports FCI alongside AGs or log shipping.

Real-world example: A legacy app requiring shared storage configures FCI on Azure VMs with clustered disks, failing over during VM maintenance without app changes.

### Configure Log Shipping

Asynchronous log backups shipped to secondaries.

- **Methods**: SSMS (wizard for primary/secondary setup); T-SQL (flexible initialization via backups).

Real-world example: A news portal configures log shipping to a warm standby server, applying logs every 15 minutes for DR, allowing quick switchover during primary failures.

### Monitor an HA/DR Solution

Ensure ongoing health.

- **Relaxed Monitoring**: Adjust heartbeat/thresholds; use T-SQL to modify lease/session timeouts, max failures; temporarily relax during high activity.

Real-world example: During peak hours, a stock trading platform relaxes AG monitoring thresholds via T-SQL to avoid false failovers from transient loads, then reverts post-peak.

### Troubleshoot an HA/DR Solution

Resolve common issues.

- **Deployment Errors**: Fix naming (AccountNameInvalid), image settings (ImageNotFound), uniqueness (InvalidResourceLocation).
- **Connectivity/Other**: Address reconfiguration, firewalls, timeouts, design flaws; apply best practices for FCI/AG; monitor resource limits.
- **Login Issues**: Check credentials in error messages.

Real-world example: Troubleshooting a hybrid AG connectivity failure due to firewall rules, an admin uses best practices to adjust timeouts and add exceptions, preventing unexpected failovers in a cross-region setup.
