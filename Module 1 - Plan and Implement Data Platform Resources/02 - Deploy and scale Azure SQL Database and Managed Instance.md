# Deploy and scale Azure SQL Database and Managed Instance

> **Exam skills:** Deploy database offerings on selected platforms, Configure Azure SQL Database for scale and performance, Configure Azure SQL Managed Instance for scale and performance, Choose an automated deployment method
> **Reading time:** ~10 min

## The big idea
When you create a PaaS database you make three choices. How you **pay** (DTU or vCore). Which **tier** (General Purpose, Business Critical, Hyperscale). How you **share** compute (one database, or a pool). Scaling later means changing these choices.

Analogy: renting a car. A DTU is a fixed package ("small car"). A vCore lets you pick engine and boot size yourself. An elastic pool is a car pool that several people share.

## Deployment models for SQL Database
- **Single database:** one database with its own compute. Billed and scaled per database.
- **Elastic pool:** many databases share one pot of compute and storage.
- Both sit on a **logical server**. It is a name and login point (`myserver.database.windows.net`), not a real machine. Firewall rules live on it.

## Purchasing models
**DTU model** (Database Transaction Unit, a fixed mix of CPU, memory and IO):
- Tiers: **Basic, Standard, Premium**.
- Simple, but you can't tune CPU and storage separately.
- **Not available on Managed Instance.**

**vCore model** (you choose vCores, memory follows, storage is separate):
- Tiers: **General Purpose, Business Critical, Hyperscale**.
- Works with **Azure Hybrid Benefit** (reuse your SQL Server licences to save money).
- Recommended for most new work.

| vCore tier | Good for | Storage | Max size |
|------------|----------|---------|----------|
| General Purpose | Normal workloads, best price | Remote premium storage | 4 TB |
| Business Critical | Low latency, high IO | Local SSD, built-in readable replica, In-Memory OLTP | 4 TB |
| Hyperscale | Big or fast-growing databases; Microsoft's **recommended default** for new OLTP | Separate storage layer with local SSD cache | **128 TB** |

**Example:** A small charity site needs a cheap, simple database. Standard S2 (DTU) is fine. A bank's trading database needs low latency. Choose Business Critical (vCore).

## Serverless compute
**Serverless** scales compute up and down by itself and bills per second.
- Available in **General Purpose** and **Hyperscale**.
- **Auto-pause** when idle: you pay only for storage while paused. Auto-pause is GA in General Purpose, preview in Hyperscale.
- Auto-pause delay: **15 min to 7 days** in General Purpose (60 min minimum in Hyperscale). Set **-1** to turn auto-pause off.
- These **block auto-pause** (auto-scale still works): geo-replication or failover groups, LTR, a DNS alias, a Data Sync database, an elastic jobs job database.
- The other option is **provisioned** compute: fixed size, billed hourly.

**Example:** A school's test database is used only in class hours. Serverless with a 1-hour auto-pause saves money at night and weekends.

## Elastic pools
An **elastic pool** shares compute between databases. A busy database borrows spare capacity from quiet ones.
- Best for **many databases with uneven, non-overlapping peaks**. Classic case: a SaaS app with one database per customer.
- DTU pools (eDTUs) or vCore pools (General Purpose, Business Critical, **Hyperscale**). You pay for the pool, not per database.
- Set **min and max per database** so one noisy database can't starve the others.
- Resize the pool from **Configure**. It is mostly online, but connections may drop briefly.
- **Hyperscale elastic pools** (GA): up to 100 TB per pool and up to 4 high-availability replicas. Adding a non-Hyperscale database converts it to Hyperscale.

**Example:** A gym software company has 200 small gym databases. Some gyms are busy at 6 AM, others at 7 PM. One pool is much cheaper than 200 separate databases.

## Hyperscale
- Compute and storage are **separate**. Storage grows on its own from 10 GB to **128 TB**.
- **Backups and restores take minutes**, whatever the size. They use storage snapshots, not full/diff/log backups.
- **Fast scaling** of compute.
- **Read scale-out:** up to **4 HA replicas**, up to **30 named replicas** (each with its own size), plus geo-replicas.
- Send reads to a replica with `ApplicationIntent=ReadOnly` in the connection string.
- **Going back:** you can return to General Purpose only **within 45 days** of moving to Hyperscale. To reach Business Critical, go through General Purpose.
- Pricing (since Dec 2023): no SQL licence fee in the price. **Azure Hybrid Benefit is not available for new Hyperscale databases.**

**Example:** An online shop's order database will reach 30 TB in two years. The team does not want to change app code. Move it to Hyperscale.

## Scaling SQL Database: quick rules
- **Scale up/down:** change tier or vCores/DTUs. Mostly online, short reconnect at the end.
- **Read scale-out:** offload reports to a readable replica (Business Critical, Premium, Hyperscale).
- **Scale out:** sharding across many databases (lesson 4).
- During a migration, scale up for speed, then scale down after.

## Deploy SQL Database: methods
| Method | Use it when |
|--------|-------------|
| Azure portal | One-off, manual setup |
| PowerShell (`New-AzSqlServer`, `New-AzSqlDatabase`, `New-AzSqlServerFirewallRule`) | Windows admins, scripts |
| Azure CLI (`az sql server create`, `az sql db create`) | Bash, Cloud Shell, Linux/Mac |
| ARM template or Bicep | Repeatable, large deployments (**declarative**: you describe the end state) |

PowerShell and CLI are **imperative** (you list the steps). Module 4 covers templates and scripts in depth.

```bash
az sql server create -g rg-shop -n shopsrv -l westeurope -u sqladmin -p '<pwd>'
az sql db create -g rg-shop -s shopsrv -n orders --edition GeneralPurpose --compute-model Serverless --family Gen5 --capacity 2
```

## Managed Instance: tiers and options
| Option | Key facts |
|--------|-----------|
| **General Purpose** | Remote storage. Standard workloads. |
| **Next-gen General Purpose** | Upgrade of GP on Elastic SAN storage. Up to **500 databases**, **32 TB**. Extra IOPS and memory set with a slider. Same base price as GP. |
| **Business Critical** | Local SSD, built-in readable secondary, **In-Memory OLTP**, faster failover. |
| **Instance pools** | Many small instances share one VM. **GP only**. Allows **2-vCore** instances. Up to 40 instances and 500 databases per pool. |
| **Stop/start** | GP only. No compute or licence charges while stopped; storage is still billed. Can be scheduled. |
| **Free offer** | One free GP instance per subscription for 12 months (4 vCores, 720 vCore hours per month, 64 GB). |

Other MI facts:
- Each database can use the whole instance storage: up to **16 TB** (GP, and Business Critical on premium-series in major regions), **32 TB** on Next-gen GP.
- **99.99% SLA**. Business Critical uses Always On replicas inside.
- Lives in a **dedicated subnet** in your VNet. Optional **public endpoint on port 3342**.
- Scale vCores and storage from the portal. It can take a while, and ends with a short failover.

**Update policy** (Settings → Maintenance and updates) decides the engine version line:
| Policy | Meaning |
|--------|---------|
| **SQL Server 2022** | Can restore databases back to SQL Server 2022; MI link failback with 2022 |
| **SQL Server 2025** | Default for new instances in the portal; restore and link with 2025 |
| **Always-up-to-date** | Newest features; **no restore back** to boxed SQL Server |

You can only move up: 2022 → 2025 → Always-up-to-date. Never down. Check it with `SELECT SERVERPROPERTY('ProductUpdateType');` → `CU` or `Continuous`. PowerShell and CLI still default to SQL Server 2022, so set the database format in scripts.

**Example:** A team may need to move a database back to on-premises SQL Server 2022 one day. Keep the MI on the **SQL Server 2022** policy.

## Backups (short version)
- Automatic full (weekly), differential (every 12 h), log (every 5–10 min).
- Point-in-time restore **1–35 days** (default 7). Long-term retention up to **10 years**.
- Restore always creates a **new** database.
- MI also allows manual **COPY_ONLY** backups to Blob Storage.
- Full detail is in the high availability and disaster recovery module.

## Remember for the exam
- **DTU is not available on Managed Instance.**
- Serverless = General Purpose and Hyperscale. Auto-pause is blocked by geo-replication, failover groups, LTR and similar features.
- Hyperscale: **128 TB**, minutes-long restores, up to **30 named replicas**, back to GP only within **45 days**.
- Elastic pool = many databases, peaks at different times. Set per-database min/max.
- MI **instance pools** = GP only, allow 2-vCore instances.
- MI **stop/start** = GP only.
- MI **Always-up-to-date** policy = no way back to SQL Server 2022/2025 restores.
- ARM/Bicep = declarative and repeatable. Portal = not repeatable.

## Quick check
1. A dev database is idle every night and weekend. How do you cut cost with least effort?
   <details><summary>Answer</summary>Use the serverless tier with auto-pause. You pay only for storage while it is paused.</details>
2. 150 tenant databases have short spikes at different times. What do you deploy?
   <details><summary>Answer</summary>An elastic pool. The databases share compute because their peaks don't overlap.</details>
3. You moved a database to Hyperscale 60 days ago. Can you go back to General Purpose?
   <details><summary>Answer</summary>No. Reverse migration only works within 45 days.</details>
