# Azure SQL High Availability, Disaster Recovery, and Backup: A Comprehensive Guide

This guide consolidates strategies for implementing **High Availability and Disaster Recovery (HADR)** and **backup/restore** processes for **Azure SQL Database**, **Azure SQL Managed Instance**, and **SQL Server on Azure Virtual Machines (VMs)**. It provides actionable insights for database administrators and architects to ensure business continuity, minimize downtime, and protect data in cloud and hybrid environments.

## Why HADR and Backups Matter in Azure SQL

HADR and backups are critical for ensuring business continuity in Azure SQL. Unlike on-premises setups where you control every aspect, Azure offers a spectrum of managed and manual solutions. Designing an effective HADR strategy requires aligning with business needs, specifically **Recovery Time Objective (RTO)** (how quickly you need to recover) and **Recovery Point Objective (RPO)** (how much data loss is tolerable). Backups act as a safety net for human errors, ransomware, or outages, ensuring you can recover critical data without significant loss.

### Real-World Analogy

HADR in Azure is like building a dream car: **IaaS** (SQL Server on VMs) lets you customize every bolt, while **PaaS** (Azure SQL Database/Managed Instance) hands you a pre-built Tesla with autopilot. Backups are like a parachute—essential for surviving crashes like accidental data deletion or regional outages. For example, a retail company with an e-commerce platform needs an **RTO of 10 minutes** (to avoid ₹50,000 in lost sales) and an **RPO of 5 minutes** (to prevent delivery and inventory issues), requiring a tailored HADR and backup strategy.

## High Availability and Disaster Recovery (HADR)

HADR ensures your database remains available and recoverable during failures, from server crashes to regional outages.

### Key Concepts: RTO and RPO

| Concept | Definition                      | Example Scenario                                           |
| ------- | ------------------------------- | ---------------------------------------------------------- |
| **RTO** | Time to recover after a failure | App must be back online within 10 minutes after a crash.   |
| **RPO** | Acceptable data loss            | Backup must cover data up to 5 minutes before the failure. |

**Example**: If backups run hourly but your RPO is 5 minutes, your strategy fails to meet business needs. Adjust to frequent log backups to minimize data loss.

### HADR Options in Azure

| Scenario                                 | Best Tool                              | Use Case                                    |
| ---------------------------------------- | -------------------------------------- | ------------------------------------------- |
| Full instance protection                 | **Failover Cluster Instance (FCI)**    | Legacy apps needing instance-level recovery |
| Fast failover + readable replicas        | **Always On Availability Groups (AG)** | High availability with read scale-out       |
| Cost-effective, manual DR                | **Log Shipping**                       | Budget-conscious disaster recovery          |
| Infrastructure-level VM failover         | **Azure Site Recovery (ASR)**          | Broad VM-level DR, not SQL-specific         |
| Multi-region DR                          | **Distributed AG**                     | Global apps needing cross-region resilience |
| PaaS automated HA/DR                     | **Auto-Failover Groups (AFG)**         | Managed failover for Azure SQL DB/MI        |
| PaaS manual DR with readable secondaries | **Active Geo-Replication**             | Reporting and DR for Azure SQL Database     |

### IaaS vs. PaaS HADR

| Type                 | Management Responsibility              | Pros                                   | Cons                              |
| -------------------- | -------------------------------------- | -------------------------------------- | --------------------------------- |
| **IaaS (VM)**        | You manage OS, SQL, HADR, backups      | Full control, supports legacy features | Complex setup, manual maintenance |
| **PaaS (SQL DB/MI)** | Azure manages HA/DR, patching, backups | Simplified, automated, stable          | Limited customization             |

**Example**:

- **IaaS**: A banking app on Azure VMs uses **Always On AG** with a **Windows Server Failover Cluster (WSFC)** and **Azure Load Balancer** for HA across Availability Zones, with a **Cloud Witness** for quorum.
- **PaaS**: A SaaS learning platform uses **Azure SQL Database** with **Auto-Failover Groups** for automatic cross-region failover, requiring no manual intervention.

### IaaS HADR Options

1. **Always On Availability Groups (AG)**:
   - Protects specific databases, supports readable secondaries.
   - **Setup**: WSFC, AG, Azure Load Balancer for listener, sync via Azure VNet.
   - **Example**: A financial ERP system syncs databases across two Azure VMs in different zones, using secondaries for reporting.
2. **Failover Cluster Instances (FCI)**:
   - Protects the entire SQL Server instance, ideal for legacy apps (e.g., SSRS, DTC).
   - **Setup**: WSFC, shared storage (Storage Spaces Direct or Premium Azure Disks), Distributed Network Name (DNN) for Server 2019+.
   - **Example**: A manufacturing app uses FCI for instance-level failover with minimal reconfiguration.
3. **Distributed Availability Groups**:
   - Syncs AGs across regions for multi-region DR.
   - **Example**: A stock trading platform syncs an AG in East US with another in West US, each with its own WSFC and listener.
4. **Azure Site Recovery (ASR)**:
   - Replicates entire VMs for infrastructure-level DR, not SQL-aware.
   - **Example**: A retail company uses ASR to replicate 10 VMs (SQL, app servers) to another region for quick recovery after a ransomware attack.

**Azure-Specific Considerations for IaaS**:
| Component | Azure Adjustment |
|-----------------|-----------------------------------------------|
| **Witness** | Use **Cloud Witness** instead of disk/file. |
| **Networking** | Use Azure DHCP, set IPs in Azure Portal. |
| **Quorum** | Critical for stability; Cloud Witness ensures single truth. |
| **AD** | Optional with Workgroup Clusters (Server 2019+). |

### PaaS HADR Options

1. **Active Geo-Replication** (Azure SQL Database):
   - Asynchronous replication to up to 4 secondary regions, manual failover.
   - Supports readable secondaries for reporting.
   - **Example**: An e-commerce app replicates its primary database in East US to West Europe for reporting and DR.
2. **Auto-Failover Groups (AFG)** (Azure SQL Database/Managed Instance):
   - Automatic failover across regions, supports multiple databases.
   - Uses read-write and read-only listeners for seamless app connectivity.
   - **Example**: An HR SaaS app groups payroll and expense databases, failing over together to another region without changing connection strings.
   - **Note**: AFG is the only DR option for Managed Instance (no Geo-Replication).

### Hybrid DR

- **Scenario**: On-premises SQL Server with DR in Azure.
- **Strategy**:
  - Use **Always On AG** to sync a secondary in Azure.
  - Store backups in **Azure Blob Storage** for cost-effective retention.
  - Use **ExpressRoute** or VPN for fast, secure sync (avoid public internet).
- **Example**: A manufacturing company syncs an on-premises SQL Server to an Azure VM, with backups in Blob Storage for long-term retention.

### Monitoring HADR

- **Azure Service Health**: Alerts on regional outages.
- **Replica Lag**: Check with `sys.dm_database_replica_states`.
- **Failover Reasons**: Review in Azure Resource Health.
- **Backup Status**: Monitor via Azure Portal, CLI, or SSMS.
- **Example**: A retail app monitors replica lag to ensure secondaries are within 5 seconds of the primary, meeting its RPO.

## Backup and Restore in Azure SQL

Backups are essential for recovering from human errors, ransomware, app bugs, or regional outages.

### Why Backups Are Critical

- **Scenarios**:
  - Accidental `DELETE FROM Users` in production.
  - Ransomware locks down an Azure VM.
  - Regional data center outage.
  - App migration corrupts data.
- **Impact**: Without backups, data loss can lead to lost revenue, compliance violations, and customer dissatisfaction.

### IaaS Backup Options (SQL Server on VMs)

You manage backups manually, as Azure doesn’t automate them for IaaS.
| Backup Type | Description | Use Case |
|---------------------|----------------------------------------|-----------------------------------|
| **Full** | Complete database snapshot | Nightly at 2 AM |
| **Differential** | Changes since last full backup | Every 6 hours |
| **Transaction Log** | Committed changes, enables log truncation | Every 15 minutes for low RPO |

**Recovery Models**:
| Model | Point-in-Time Restore | Use Case |
|-----------------|-----------------------|----------------------------------------|
| **FULL** | Yes | Critical apps needing minimal data loss |
| **SIMPLE** | No | Dev/test environments |
| **BULK_LOGGED** | Partial | Bulk operations with speed priority |

**Backup Methods**:

1. **Azure VM Snapshot**:
   - Captures entire VM, ideal for ransomware or disaster recovery.
   - Not suitable for granular database restores.
   - **Fix for Issues**: Add registry key:
     ```reg
     [HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\BCDRAGENT]
     "USEVSSCOPYBACKUP"="TRUE"
     ```
2. **SQL Native Backups to Disk/Azure Files**:
   - Write `.bak` files to local disk or Azure Files, copy to offsite storage (e.g., Blob).
   - **Example**: Daily backups to Azure Files, mirrored to Blob Storage for redundancy.
3. **Backup to Azure Blob (URL Backup)**:
   - Native SQL Server support for cloud storage.
   - **Example**:
     ```sql
     BACKUP DATABASE contoso
     TO URL = 'https://myacc.blob.core.windows.net/db/contoso.bak'
     ```
   - Restore with `RESTORE DATABASE FROM URL`.
4. **SQL IaaS Agent Extension**:
   - Automates full, differential, and log backups to Azure Blob.
   - Includes retention policies.
   - **Warning**: Avoid mixing with manual backups to preserve the log chain.

**Restore Process**:

- Use SQL scripts or SSMS for restores (no automated "Restore" button).
- Requires full backup, differential (if any), and log backups up to the desired point-in-time.

### PaaS Backup and Restore (Azure SQL Database/Managed Instance)

Azure automates backups with geo-redundant storage (RA-GRS).

- **Backup Schedule**:
  - Full: Weekly.
  - Differential: Every 12 hours.
  - Transaction Log: Every 5–10 minutes.
- **Restore Options**:
  | Action | Available | Details |
  |------------------------|-----------|---------------------------------------|
  | Point-in-Time Restore | Yes | Any point within 7–35 days. |
  | Restore Deleted DB | Yes | Requires a new database name. |
  | Restore in Place | No | Creates a new database. |
  | Manual Backup to URL | Yes (MI) | Only for Managed Instance. |

**Example**: A SaaS app accidentally deletes a table. Using the Azure Portal, the DBA restores the database to a point 10 minutes before the deletion, creating a new database for recovery.

### Backup Best Practices

- **IaaS**: Manage backups manually; use **SQL IaaS Agent Extension** for automation.
- **PaaS**: Leverage Azure’s automated backups but understand restore processes.
- **Log Chain**: Avoid mixing backup methods to maintain a consistent transaction log chain.
- **Offsite Storage**: Copy backups to **Azure Blob Storage** for geo-resilience.
- **Testing**: Regularly simulate restores to validate recovery plans.

## Real-World Scenario

A **retail e-commerce platform**:

- **HADR**: Uses **Auto-Failover Groups** for its Azure SQL Database, ensuring automatic failover to a secondary region with an RTO of 10 minutes and RPO of 5 minutes. Readable secondaries handle reporting.
- **Backup**: Relies on Azure’s automated backups for point-in-time restores. For long-term retention, exports monthly backups to Blob Storage.
- **Monitoring**: Tracks replica lag with `sys.dm_database_replica_states` and sets Azure Monitor alerts for failover events.
- **Recovery**: When a developer accidentally deletes order data, the DBA restores a database to 5 minutes prior using the Azure Portal, minimizing disruption.

## Key Takeaways

- **HADR Design**: Align with **RTO** and **RPO** based on business needs (e.g., 10-minute recovery, 5-minute data loss).
- **IaaS Options**:
  - **Always On AG**: Database-level HA with readable secondaries.
  - **FCI**: Instance-level protection for legacy apps.
  - **Distributed AG**: Multi-region DR.
  - **ASR**: VM-level DR for broad infrastructure protection.
- **PaaS Options**:
  - **Active Geo-Replication**: Manual DR with readable secondaries.
  - **Auto-Failover Groups**: Automated failover for SQL Database/MI.
- **Hybrid DR**: Sync on-premises SQL to Azure VMs with AG and store backups in Blob Storage.
- **Backups**:
  - **IaaS**: Use **SQL IaaS Agent Extension** or **URL backups** to Blob; avoid mixing methods.
  - **PaaS**: Automated backups with point-in-time restores (7–35 days).
- **Monitoring**: Use **Azure Service Health**, **Resource Health**, and DMVs to track HADR and backup status.
- **Testing**: Regularly test failovers and restores to ensure reliability.

By combining the right HADR tools and a robust backup strategy, you can protect Azure SQL environments from outages, ensure compliance, and maintain business continuity for applications like e-commerce or financial systems.
