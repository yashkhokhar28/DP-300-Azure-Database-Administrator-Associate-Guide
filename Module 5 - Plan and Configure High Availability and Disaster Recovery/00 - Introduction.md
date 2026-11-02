# Module 5: Plan and configure a high availability and disaster recovery (HA/DR) environment

**Exam weight:** 20–25% · **Reading time:** ~71 min · **Files:** 6

## What you will learn
You will learn how to keep a database running when something breaks. You will learn how to bring it back after a big disaster. You will pick the right tool for each Azure SQL option: SQL Server on a VM, Azure SQL Database and Azure SQL Managed Instance. You will learn to back up and restore databases, with T-SQL and with Azure's automatic backups. Finally, you will learn how to test, watch and fix an HA/DR setup.

Think of a busy hospital. If one generator fails, a second one starts at once and the lights stay on. That is **high availability**. If the whole building floods, staff move to a second hospital in another city. That is **disaster recovery**. Every night the hospital also copies all patient records to a safe archive. That is **backup**. The hospital runs fire drills to prove all of this really works. That is **testing**.

## Lessons in this module
| # | Lesson | What it covers | Time |
|---|--------|----------------|------|
| 1 | [HA/DR basics - RPO, RTO and choosing a strategy](<01 - HA-DR basics - RPO, RTO and choosing a strategy.md>) | RPO, RTO, HA vs DR, IaaS vs PaaS, picking a tool | ~9 min |
| 2 | [HA/DR for SQL Server on Azure VMs and hybrid](<02 - HA-DR for SQL Server on Azure VMs and hybrid.md>) | Clusters, FCI, availability groups, log shipping, Site Recovery, hybrid | ~17 min |
| 3 | [HA/DR for Azure SQL Database and Managed Instance](<03 - HA-DR for Azure SQL Database and Managed Instance.md>) | Built-in HA, zones, geo-replication, failover groups, MI link | ~12 min |
| 4 | [Backup and restore](<04 - Backup and restore.md>) | T-SQL backup/restore, backup to URL, automated backups, PITR, LTR, geo-restore | ~15 min |
| 5 | [Monitor, test and troubleshoot HA/DR](<05 - Monitor, test and troubleshoot HA-DR.md>) | DR drills, failover commands, DMVs, common errors | ~13 min |

## Words you will meet
| Word | Simple meaning |
|------|----------------|
| HA (high availability) | Stay online when one part fails, usually in the same region |
| DR (disaster recovery) | Get back online after a big failure, usually in another region |
| RTO (recovery time objective) | Most downtime the business can accept |
| RPO (recovery point objective) | Most data (measured in time) the business can afford to lose |
| Failover | Switching work from the broken copy to a healthy copy |
| Replica | A copy of a database kept up to date |
| Synchronous / asynchronous | Wait for the copy to confirm / do not wait |
| WSFC | Windows Server Failover Cluster, the Windows feature that manages failover |
| FCI | Failover cluster instance, protects a whole SQL Server instance |
| AG | Always On availability group, protects a set of databases |
| Listener | One fixed name apps connect to, whichever copy is primary |
| Failover group | Azure feature that fails over a group of databases with a listener |
| PITR | Point-in-time restore, restore to an exact minute |
| LTR | Long-term retention, keep backups for up to 10 years |

## Exam skills covered
- [ ] Recommend HA/DR strategy based on RPO/RTO requirements → 1
- [ ] Evaluate HA/DR for hybrid deployments → 2, 3
- [ ] Evaluate Azure-specific HA/DR solutions → 1, 2, 3
- [ ] Plan a testing procedure for an HA/DR solution → 5
- [ ] Recommend a database backup and restore strategy → 4
- [ ] Perform a database backup by using native tools → 4
- [ ] Perform a database restore by using native tools → 4
- [ ] Perform a database restore to a point in time → 4
- [ ] Configure long-term backup retention → 4
- [ ] Backup and restore a database by using T-SQL → 4
- [ ] Backup to and restore from cloud storage → 4
- [ ] Configure active geo-replication → 3
- [ ] Configure Always On availability groups on SQL Managed Instance and Azure VMs → 2, 3
- [ ] Configure failover groups → 3
- [ ] Configure Always On Failover Cluster Instances on Azure VMs → 2
- [ ] Configure log shipping → 2
- [ ] Monitor an HA/DR solution → 5
- [ ] Troubleshoot an HA/DR solution → 5
