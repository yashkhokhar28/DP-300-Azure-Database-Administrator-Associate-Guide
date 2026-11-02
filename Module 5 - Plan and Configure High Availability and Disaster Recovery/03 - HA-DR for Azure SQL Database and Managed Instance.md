# HA/DR for Azure SQL Database and Managed Instance

> **Exam skills:** Evaluate Azure-specific HA/DR solutions, Evaluate HA/DR for hybrid deployments, Configure active geo-replication, Configure failover groups, Configure Always On availability groups on SQL Managed Instance and Azure VMs
> **Reading time:** ~12 min

## The big idea
In PaaS, Microsoft already handles HA inside a region. You don't build clusters. Your job is to choose extra protection: **zone redundancy** for datacenter failures, and **active geo-replication** or **failover groups** for regional failures. For hybrid, the **Managed Instance link** connects SQL Server to SQL MI.

Think of renting a flat. The landlord fixes the boiler (built-in HA). You decide whether to also rent a holiday home in another city in case of a flood (geo-replication or failover groups).

## Built-in high availability
- Azure SQL Database SLA: **99.99%**. Zone-redundant Business Critical/Premium: **99.995%**.
- If hardware fails, Azure starts a new node and reattaches storage, or switches to another replica. This is automatic.
- During failover, **open connections drop** and **in-progress transactions roll back**. Apps must use **retry logic** (try again after a short wait).
- Business Critical/Premium keep several replicas inside the region. One can be used for read-only queries (**read scale-out**).

**Accelerated Database Recovery (ADR)** makes rollback and crash recovery very fast, even for long transactions. It is **on by default in PaaS and can't be turned off**. It is also in SQL Server 2019+.

Azure also runs hidden health checks: automatic **DBCC CHECKDB**-style integrity checks, lost-write and stale-read detection, and **automatic page repair** of damaged pages.

PaaS differences from a VM: you **can't** set a database OFFLINE or EMERGENCY, and failover settings are limited to what Azure offers.

## Zone redundancy
- Spreads the database's replicas across **availability zones** in one region. Survives a datacenter failure. Transparent to apps.
- **SQL Database:** General Purpose, Business Critical, Hyperscale and Premium. **Not Basic or Standard.**
- **SQL Managed Instance:** General Purpose and Business Critical.

**Example:** A school app on Business Critical turns on zone redundancy. If one datacenter in the region loses power, the app keeps working after a short reconnect.

## Active geo-replication (Azure SQL Database only)
- Creates **readable secondary databases**, usually in **other regions**, updated **asynchronously** (some data loss is possible).
- Uses AG technology behind the scenes.
- Up to **4 readable geo-secondaries** per primary, in the same or different regions. Chain them for more.
- Works for single databases and elastic pools, including **Hyperscale**. **Not supported on SQL Managed Instance.**
- The secondary must use the **same service tier**. Same **compute size** and backup redundancy are strongly recommended. A smaller secondary causes **lag** with heavy writes.
- Failover is **manual** (portal, PowerShell, CLI, T-SQL). After failover the **server name changes**, so apps must **update connection strings**.
- **Cross-subscription:** same Microsoft Entra tenant → portal works. Different tenant → **SQL authentication + T-SQL** (or REST API). Not supported with Microsoft Entra-only authentication when an Entra user creates it.

**Configure it:** Portal → the database → **Data management → Replicas** → **Create replica** → pick the server in another region. Or:

```bash
az sql db replica create -g rg-prod -s sql-prod-weu -n SalesDb --partner-server sql-prod-neu --partner-resource-group rg-dr
```

```sql
-- In master of the primary server
ALTER DATABASE [SalesDb] ADD SECONDARY ON SERVER [sql-prod-neu] WITH (ALLOW_CONNECTIONS = ALL);
```

**Example:** A travel site in East US adds a geo-secondary in West US for reports and DR. During a storm, the DBA fails over manually and updates the app to the new server name.

## Failover groups
**Failover groups** (formerly "auto-failover groups") work on **SQL Database and SQL Managed Instance**.
- Replicate **a group of databases** to a server or instance in **another region** and fail them over **together**.
- Give two **listeners** (DNS names that always point to the right place):
  - Read-write: `<fog-name>.database.windows.net`
  - Read-only: `<fog-name>.secondary.database.windows.net` (use `ApplicationIntent=ReadOnly`)
  - MI uses `<fog-name>.<zone_id>.database.windows.net`.
- After failover, **no connection string change** is needed.
- Replication is **asynchronous**.

Scope:
- **SQL Database:** some or all databases on a server. Many groups per server.
- **SQL MI:** **all user databases**, **one group per instance**. **System databases are not replicated** (logins and Agent jobs in master/msdb must be created on the secondary yourself).

### Failover policy
| Policy | Value in CLI/PowerShell | What happens |
|---|---|---|
| **Customer managed (recommended)** | `Manual` | You decide when to fail over (planned or forced). Use it to test and run your DR plan |
| **Microsoft managed** | `Automatic` | Microsoft forces failover of **all** such groups in the region, only for a **wide regional outage**, after a **grace period** (min and default **1 hour**). Never for one group. Don't rely on it |

- `GracePeriodWithDataLossHours` = how long Azure waits before a forced failover (possible data loss). Set it higher if you can't afford data loss.
- `az sql failover-group create` now defaults to **Manual**.
- **Read-only listener failover** is **off by default**: if the secondary is offline, read-only traffic is **not** sent to the primary (to protect primary performance). You can turn it on.
- **Failback is always manual.**

### Configure a failover group
1. Create a secondary server (SQL DB) or instance (MI) in another region.
2. Create the failover group (portal: server → **Failover groups** → **Add group**, or CLI/PowerShell) and add databases.
3. Azure **seeds** (makes the first full copy of) the secondary databases.
4. Point apps at the **listener** name.

```bash
az sql failover-group create --name fog-sales -g rg-prod --server sql-prod-weu \
  --partner-server sql-prod-neu --add-db SalesDb --failover-policy Manual
```

**MI requirements:** both instances in the **same DNS zone** (set `DNSZonePartner` when creating the secondary), connected by **global VNet peering** (recommended), VPN or ExpressRoute.

**Example:** An HR SaaS app has payroll and expenses databases. A failover group moves both together to North Europe. The app keeps using `fog-hr.database.windows.net`.

## License-free standby replica
If a secondary is used **only for DR** (no reads), mark it as a **standby** replica. You pay **no SQL Server licence** for its vCores. Compute and storage are still billed. Works for SQL DB geo-secondaries and SQL MI failover groups.

### Connection strings with a failover group
- Read-write apps: `Server=tcp:fog-sales.database.windows.net;...`
- Report apps: `Server=tcp:fog-sales.secondary.database.windows.net;ApplicationIntent=ReadOnly;...`
- Don't use the server names (`sql-prod-weu...`) in apps. After a failover they point to the wrong role.
- Logins: use **contained database users** (users stored in the database, so they move with it). For server logins on SQL DB, create the same login with the **same SID** on the secondary server.

## PaaS HA/DR scenarios
| Scenario | Pick |
|---|---|
| Survive a single node failure | Nothing to do: built-in HA |
| Survive a datacenter failure in the region | **Zone redundancy** |
| Offload reports in the same region (Business Critical) | **Read scale-out** replica (`ApplicationIntent=ReadOnly`) |
| Readable copies in several regions for one database | **Active geo-replication** |
| Several databases must fail over together, same connection string | **Failover group** |
| Cross-region DR for SQL MI | **Failover group** |
| DR between on-premises SQL Server 2022 and SQL MI with failback | **Managed Instance link** |
| No replica at all, region lost | **Geo-restore** from backups (lesson 4) |

**Example:** A food delivery app runs on Azure SQL Database Business Critical with zone redundancy (HA). A failover group to the paired region gives DR. Reports read from the read-only listener. The secondary is marked as a standby replica only if no reports use it.

## Geo-replication vs failover groups
| | Active geo-replication | Failover group |
|---|---|---|
| SQL Database | Yes | Yes |
| SQL Managed Instance | **No** | **Yes** (only geo-DR option for MI) |
| Unit | One database | Group of databases |
| Number of secondaries | Up to 4 | One (multiple is preview for SQL DB) |
| Same region allowed | Yes | No, cross-region only |
| Connection string after failover | **Changes** | **Stays the same** (listener) |
| Automatic failover | No | Only with Microsoft-managed policy |
| Readable secondary | Yes | Yes |

## Always On AGs and SQL Managed Instance: the MI link
You can't build your own AG inside SQL MI. Built-in HA already uses AG technology. To connect SQL Server and SQL MI, use the **Managed Instance link**.
- It uses a **distributed availability group** under the hood. Each link covers **one database**.
- **SQL Server 2016 SP3, 2017 CU31, 2019 CU20**: **one-way** to MI. Good for migration, read replicas and DR. Failing over to MI breaks the link. No failback.
- **SQL Server 2022 and 2025**: **two-way**. Manual **failover and failback**, online. The MI **update policy** must match (SQL Server 2022 or 2025, not "Always-up-to-date").
- Editions: Enterprise, Standard, Developer.
- Traffic uses the MI **VNet-local endpoint**, so you need VPN, ExpressRoute or VNet peering.
- A DR-only MI replica can be **licence-free** (hybrid failover rights).
- An MI with a link **can't be in a failover group** (and vice versa).

**Example:** A bank runs SQL Server 2022 on-premises. It links its main database to SQL MI in Azure as a DR copy. During a datacenter outage it fails over to MI, then later fails back.

## Remember for the exam
- PaaS HA is **automatic**. Apps need **retry logic**.
- Zone redundancy: not on Basic/Standard.
- **Geo-replication = SQL Database only**, up to 4 secondaries, manual failover, **server name changes**.
- **Failover group = SQL DB and MI**, listener, **no connection string change**, databases fail over together.
- MI failover group: **all user DBs**, **one group per instance**, system DBs **not** replicated, same **DNS zone**.
- Customer managed (`Manual`) policy is recommended. Microsoft managed waits at least **1 hour** and covers the whole region.
- Read-only listener failover is **off by default**.
- AG with SQL MI → **Managed Instance link** (two-way needs SQL Server 2022/2025).
- DR-only secondary → **standby** replica, no licence cost.

## Quick check
1. You need cross-region DR for SQL Managed Instance. Which feature?
   <details><summary>Answer</summary>A failover group. Active geo-replication is not available for SQL MI.</details>
2. An app must keep the same connection string after a regional failover of three Azure SQL databases. Which feature?
   <details><summary>Answer</summary>A failover group, because apps use its read-write listener and all three databases fail over together.</details>
3. You need four readable copies of one Azure SQL database in four regions for local reporting. Which feature?
   <details><summary>Answer</summary>Active geo-replication. It supports up to 4 readable secondaries. A failover group has one secondary.</details>
