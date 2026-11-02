# HA/DR basics - RPO, RTO and choosing a strategy

> **Exam skills:** Recommend HA/DR strategy based on RPO/RTO requirements, Evaluate Azure-specific HA/DR solutions
> **Reading time:** ~9 min

## The big idea
Before you pick any HA/DR tool, the business must answer two questions. How long can we be down? How much data can we lose? The answers are the **RTO** and the **RPO**. The tool you pick must meet both, at a cost the business accepts.

Think of a pizza shop. If the oven breaks, how long can the shop stay closed (RTO)? If the order book burns, how many orders can it forget (RPO)?

## RTO - recovery time objective
- **RTO** = the **longest time** the system may be down after a failure.
- Question it answers: "How long can the business afford to be down?"
- It can be set for a whole system or for one part (one instance, one database).
- The **slowest part sets the real RTO**. If the database is back in 5 minutes but the app server needs 20 minutes, the total RTO is 20 minutes.

**Example:** An online ticket site has an RTO of 30 minutes. A server fails at 9:00. The site must be working again by 9:30.

## RPO - recovery point objective
- **RPO** = the **most data, measured in time**, you may lose.
- Question it answers: "How much data loss is acceptable?"
- Your **backup or copy frequency** limits your RPO. If log backups run every 30 minutes, your RPO can't be better than 30 minutes.

**Example:** A bank app has an RPO of 15 minutes. A failure hits at 10:00. The data must be restorable to 9:45 or later. Payments made between 9:45 and 10:00 might be lost.

## How to set RTO and RPO
- **Business needs drive them.** If an hour of downtime costs $10,000, a faster (more expensive) solution is worth it.
- **Balance cost and risk.** Zero downtime and zero data loss are very expensive and often not practical.
- **Be realistic.** If a restore takes 3 hours but the RTO is 2 hours, the plan fails.
- **Write them down, test them and review them** regularly.

## HA vs DR
| | High availability (HA) | Disaster recovery (DR) |
|---|---|---|
| Protects against | One server, disk or datacenter failing | A whole site or region failing |
| Where the copy lives | Same region (same datacenter or another zone) | Another region (or on-premises to Azure) |
| Typical RTO/RPO | Seconds to minutes | Minutes to hours |
| Example | Failover to another node in the same region | Switch to a copy in another region |

**Example:** A shop app has two copies in West Europe zones (HA) and one more copy in North Europe (DR). A zone power cut uses the HA copy. A full regional outage uses the DR copy.

Backups are not HA. They are the **last line of defence** against mistakes, bugs, ransomware and disasters. You need them even when you have HA and DR.

## IaaS vs PaaS - who does the work
| | IaaS: SQL Server on Azure VM | PaaS: Azure SQL Database / Managed Instance |
|---|---|---|
| Who manages SQL Server and OS | You | Microsoft |
| HA | You build it (AG, FCI) plus availability sets/zones | Built in, automatic |
| DR | You build it (AG, distributed AG, log shipping, Site Recovery) | You turn it on (geo-replication, failover groups) |
| Backups | You set them up | Automatic |
| Control | Full | Limited, but much less work |

**Example:** A company with an old app that needs SQL Server Agent jobs and instance-level settings may pick a VM and build an AG. A new SaaS app picks Azure SQL Database and simply adds a failover group.

## Choosing a strategy
| If you need... | Pick |
|---|---|
| HA for a whole SQL Server instance (logins, jobs, everything) on VMs | **Failover cluster instance (FCI)** |
| Fast HA for some databases, with readable copies, on VMs | **Always On availability group (AG)** |
| Cheap, simple DR with manual failover | **Log shipping** |
| DR between regions or clusters for VMs | **AG across regions** or **distributed AG** |
| Simple DR for whole VMs, not SQL-aware | **Azure Site Recovery** |
| Survive a datacenter failure for VMs | **Availability zones** |
| Survive host hardware/maintenance in one datacenter | **Availability sets** |
| PaaS HA in one region | **Built-in HA**, add **zone redundancy** |
| PaaS DR, readable copies, Azure SQL Database only | **Active geo-replication** |
| PaaS DR, several databases together, no connection string change | **Failover group** |
| Fix human error (deleted rows) | **Point-in-time restore** |
| Keep backups for years (compliance) | **Long-term retention (LTR)** |

**Example:** An e-commerce site needs RPO of 5 minutes and RTO of 10 minutes on Azure SQL Database. Automatic log backups give PITR. A failover group gives fast regional DR without changing the app's connection string.

## From requirement to design: three worked cases
Read the requirement words in a question and map them to a tool.

**Case 1 - "lowest cost, a few hours of downtime is fine".** A small charity runs SQL Server on one Azure VM. RTO is 8 hours, RPO is 1 hour. Hourly log backups to Blob Storage (backup to URL) with geo-redundant storage are enough. A second always-running server would be wasted money.

**Case 2 - "near-zero data loss inside the region, automatic failover".** A hospital runs SQL Server on VMs. RPO is near zero and RTO is under a minute. Use an **AG with two synchronous replicas in two availability zones** and automatic failover. Add an async replica in another region for DR.

**Case 3 - "minimal admin effort, regional DR, apps must not change".** A SaaS company uses Azure SQL Database. Use built-in HA plus **zone redundancy**, and a **failover group** to another region. Apps use the listener, so nothing changes after failover.

Key words to watch for:
| Question says | Think |
|---|---|
| "minimise administrative effort" | PaaS built-in features |
| "no data loss" | Synchronous commit, planned failover |
| "minimise cost" | Log shipping, backups, LRS for non-critical |
| "instance-level objects" | FCI or contained AG |
| "without changing connection strings" | Failover group or AG listener |

## Remember for the exam
- **RTO = time** to come back. **RPO = data** you can lose (in time).
- Backup frequency limits RPO. Restore speed limits RTO.
- The slowest component decides the real RTO.
- HA = same region, short RTO. DR = other region, longer RTO.
- IaaS: you build HA/DR. PaaS: HA is built in, you configure DR.
- "Protect the whole instance" → FCI. "Protect some databases, readable copies" → AG.
- "Cheapest DR, manual failover is OK" → log shipping.
- "No connection string change after failover" → failover group.

## Quick check
1. Log backups run every hour. The business asks for an RPO of 10 minutes. Is the plan good?
   <details><summary>Answer</summary>No. Up to one hour of data could be lost. Take log backups at least every 10 minutes, or use a replica.</details>
2. A database recovers in 5 minutes but the web tier takes 25 minutes to restart. What is the real RTO?
   <details><summary>Answer</summary>25 minutes. The slowest part sets the RTO.</details>
3. You need HA for SQL Server on VMs in one region and must also keep logins and Agent jobs together with the data. Which SQL feature protects the whole instance?
   <details><summary>Answer</summary>A failover cluster instance (FCI). An AG protects databases only (unless it is a contained AG).</details>
