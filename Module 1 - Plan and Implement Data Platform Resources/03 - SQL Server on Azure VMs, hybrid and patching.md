# SQL Server on Azure VMs, hybrid and patching

> **Exam skills:** Deploy database offerings on selected platforms, Configure SQL Server on Azure Virtual Machines for scale and performance, Deploy hybrid SQL Server solutions, Identify use cases for Azure Arc-enabled SQL services, Apply patches and updates for hybrid and IaaS deployments
> **Reading time:** ~11 min

## The big idea
A SQL Server VM is your own server in Azure. Microsoft owns the hardware. You own the OS, SQL Server, disks, network and **patching**. Hybrid means some SQL Servers stay on-premises and work with Azure for backup, DR (disaster recovery) or management.

Analogy: renting an empty flat. The landlord fixes the building. You buy the furniture, clean and repair everything inside.

## Deploying a SQL Server VM
- Use an **Azure Marketplace** image, for example SQL Server 2022 on Windows Server 2022. SQL is already installed.
- During setup you choose SQL authentication, port, storage layout and **SQL connectivity**:
  - **Public:** reachable from the internet.
  - **Private:** only inside the virtual network.
  - **Local:** only from the same VM.
- Portal setup is not easy to repeat. Use ARM/Bicep templates for many VMs.

**Licensing options:**
| Option | Meaning |
|--------|---------|
| Pay-as-you-go | SQL licence included in the VM price, billed per second |
| Azure Hybrid Benefit / BYOL | Reuse your own SQL Server licences with **Software Assurance** |
| HA/DR licence | Free **passive** replicas for HA/DR. Switching licence type is online, no restart |
| Reserved instances | Commit 1–3 years to save on the VM |

## SQL Server IaaS Agent extension
The **SQL IaaS Agent extension** links the VM to a "SQL virtual machine" resource in the portal. It gives you:
- Automated backup and (legacy) Automated Patching.
- **Azure Key Vault** integration and Microsoft **Defender for Cloud** integration.
- Disk usage view, licence and edition changes, **best-practices assessment**.

Current facts:
- **No more management modes** (lightweight/full were removed in Mar 2023). Basic registration is free.
- The agent installs only when you turn on a feature that needs it, with least-privilege rights.
- Self-installed SQL Server 2016+ VMs are **registered automatically**.

## VM size for SQL
| VM family | Use |
|-----------|-----|
| General purpose | Dev/test, small databases |
| Compute optimized | High CPU, app servers |
| **Memory optimized** | **Most SQL workloads.** Ebdsv5 = best price-performance. M-series for very large memory |
| Storage optimized | Fast local disks for scale-out data |
| GPU / HPC | ML, rendering, special compute |

If a VM is too small, **resize** it to a bigger size. This needs a restart.

## Disks and storage
| Disk | Use | Max IOPS | Max MB/s |
|------|-----|----------|----------|
| Ultra Disk | Most demanding IO | 400,000 | 10,000 |
| Premium SSD v2 | Best price-performance for most SQL | 80,000 | 2,000 |
| Premium SSD | General production SQL | 20,000 | 900 |
| Standard SSD | Light workloads | 6,000 | 750 |
| Standard HDD | Backups, non-critical | 2,000 | 500 |

Rules:
- **C:** = OS disk. **D:** = temporary disk; data is **lost on reboot**. Good for **tempdb** only.
- Put **data** and **log** files on **separate data disks**.
- **Read caching** on data disks. **No caching** on log disks.
- Stripe several disks (Storage Spaces) for more IOPS.
- Use Premium SSD, Premium SSD v2 or Ultra for data and log. Standard storage is fine for backups.

**Good SQL settings on a VM:** backup compression on, **instant file initialization** on, system databases moved off C:, autoshrink and autoclose **off**, sensible autogrowth, **max server memory** set, **lock pages in memory**, **optimize for ad hoc workloads**, **Query Store** on, regular DBCC CHECKDB and index/statistics maintenance.

**Encryption at rest:** server-side encryption is on by default. Azure Disk Encryption (BitLocker inside the VM) **retires 15 Sep 2028**. Use **encryption at host** for new VMs.

**Example:** A payroll VM has slow writes at month end. The log file shares a cached disk with data. Move the log to its own Premium SSD v2 disk with no caching.

## HA on VMs (short version)
- **Availability zones** (~99.99%) spread VMs over data centres. **Availability sets** (~99.95%) spread them over racks.
- **Always On availability groups** (database level, up to 9 replicas) and **failover cluster instances** (instance level, shared storage).
- Full detail is in the high availability and disaster recovery module.

## Hybrid SQL Server setups
Hybrid means on-premises and Azure work together. Common setups:
1. **DR in Azure:** on-premises SQL runs daily work. An Azure VM is the standby (for example an async AG replica). You get a second site without building a second data centre.
2. **Backups to Azure:** `BACKUP TO URL` sends backups to Blob Storage. You can restore them on an Azure VM in a drill or a real disaster.
3. **Data files in Azure Storage:** SQL runs on-premises but database files sit in Azure Blob Storage.
4. **MI link:** replicate an on-premises database to Managed Instance for DR or read offload (lesson 7).
5. **Azure Arc:** manage, assess and patch on-premises SQL Servers from Azure.

Networking for hybrid:
- **Site-to-site VPN:** cheaper, goes over the internet, higher latency.
- **ExpressRoute:** private line, low latency, more expensive.
- **Point-to-site VPN:** one machine only, good for admin or testing.
- Active Directory and DNS must work in both places.

**Example:** A bank keeps its main SQL Server in its own data centre for compliance. It adds an async AG replica on an Azure VM over ExpressRoute for DR, and Arc-enables the on-premises server for one view in the portal.

## Who patches what
| Platform | Who patches | Your controls |
|----------|-------------|---------------|
| SQL Server on Azure VM | **You** | Azure Update Manager, legacy Automated Patching, manual |
| Arc-enabled SQL Server | **You** | Arc automatic updates, Azure Update Manager |
| Managed Instance | Microsoft | Maintenance window, update policy |
| SQL Database | Microsoft | Maintenance window |

## Automated Patching (legacy, retiring)
- Part of the SQL IaaS Agent extension.
- Sets a window: day, start hour, **30–180 minutes** duration.
- Installs only Windows and SQL updates marked **Important or Critical**. **No cumulative updates (CUs).**
- **Not AG-aware.** Give each replica a different day, or both may reboot together.
- **Retires 17 Sep 2027.** Don't use it for new VMs.

## Azure Update Manager (the replacement)
- One service for **Azure VMs and Arc-enabled servers**, **Windows and Linux**.
- Needs no Log Analytics or Azure Automation.
- **Installs SQL Server CUs** (Automated Patching can't).
- **Periodic assessment** every 24 hours shows missing updates.
- **One-time updates** on demand, or **scheduled updates** with a **maintenance configuration**. Supports dynamic scope and Azure Policy.
- **Hotpatching** on supported Windows Server Azure Edition VMs (no reboot for many fixes).

To switch a SQL VM to Update Manager:
1. Disable Automated Patching.
2. Turn on **Enable Microsoft updates** (needed to get SQL Server updates, not only Windows).
3. Set patch orchestration to **Customer Managed Schedules**, and include the **Updates** classification to get CUs.

Or run the official script: `Convert-SQLVMPatchingSchedule` (module MigrateSQLVMPatchingSchedule-Module). It does all of this for you.

Never use **both** tools on one VM. Update Manager is **also not AG-aware**, so put replicas in different maintenance configurations.

## Patching Arc-enabled SQL Server
- **Arc automatic updates:** weekly schedule, Important/Critical only, no CUs, **Windows only**, applies to all instances on the host.
- It needs licence type **Pay-as-you-go or Software Assurance**. It is greyed out for **License only**.
- Or use **Azure Update Manager** for Arc servers (same as Azure VMs).

## Rolling patch for an availability group
Goal: only one short outage (one manual failover).
1. Set all synchronous replicas to **manual failover**.
2. Patch **async secondaries**, then remote sync secondaries, then local sync secondaries.
3. Check the target is **SYNCHRONIZED**, then **fail over manually** to a patched sync secondary.
4. Patch the **old primary**.
5. Turn automatic failover back on. Fail back if you want.

```sql
ALTER AVAILABILITY GROUP [AG1] MODIFY REPLICA ON 'SQLVM02' WITH (FAILOVER_MODE = MANUAL);
-- after patching the secondary, run on it:
ALTER AVAILABILITY GROUP [AG1] FAILOVER;
```

- **Never patch the primary first.** A newer primary can't send log to older secondaries.
- **Cluster-Aware Updating is not supported** for AGs.
- **FCI:** patch passive nodes, fail over, then patch the old active node.

## PaaS maintenance window
Microsoft patches SQL Database and MI. You choose **when**:
| Option | Time |
|--------|------|
| System default | Big updates avoided 8 AM–5 PM local time |
| Weekday | 10 PM–6 AM, Mon–Thu |
| Weekend | 10 PM–6 AM, Fri–Sun |

- Free. Not for SQL Database Basic, S0, S1, or MI instance pools.
- **Advance notifications** 24 h before (SQL Database needs a non-default window for this).
- On MI, changing the window **changes the IP address**. Check firewall rules.
- Geo-replicas not in paired regions should use different windows.

## Remember for the exam
- "Install SQL Server **CUs** on a schedule" → **Azure Update Manager**.
- Automated Patching = Important/Critical only, no CUs, retires **17 Sep 2027**.
- "Both AG nodes rebooted together" → overlapping schedules. No patch tool is AG-aware.
- "Least effort to move off Automated Patching" → `Convert-SQLVMPatchingSchedule`.
- Arc automatic updates greyed out → licence type is **License only**.
- Temporary D: drive is wiped on reboot. Logs on uncached disks, data on read-cached disks.
- "Stop patches during business hours on SQL Database or MI" → **maintenance window**.

## Quick check
1. You must install SQL Server cumulative updates on 50 SQL VMs every month. Which tool?
   <details><summary>Answer</summary>Azure Update Manager. Automated Patching does not install CUs.</details>
2. Arc automatic updates is greyed out for a server. Why?
   <details><summary>Answer</summary>Its licence type is License only. Change it to pay-as-you-go or Software Assurance.</details>
3. In an AG rolling update, which replica do you patch last?
   <details><summary>Answer</summary>The original primary, after a manual failover to a patched synchronous secondary.</details>
