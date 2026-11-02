# Module 1: Plan and Implement Data Platform Resources

**Exam weight:** 15–20% · **Reading time:** ~88 min · **Files:** 10

## What you will learn
You will learn the three main ways to run SQL Server in Azure, and how to pick one. You will learn how to size and scale each one. You will learn how to split big tables (partitioning), shrink them (compression) and spread data over many databases (sharding). Finally, you will learn how to move existing databases into Azure and fix problems on the way.

Think of moving a family into a new city. First you choose a home: your own house (a VM), a serviced flat (Managed Instance) or a hotel room (SQL Database). Then you pick the right size and arrange the rooms. Then you plan the move: pack everything in one weekend (offline), or move room by room while you still live there (online). Last, you deal with boxes that got lost.

## Lessons in this module
| # | Lesson | What it covers | Time |
|---|--------|----------------|------|
| 1 | [Choose the right Azure SQL option](<01 - Choose the right Azure SQL option.md>) | VM vs Managed Instance vs SQL Database, Arc, Fabric | ~10 min |
| 2 | [Deploy and scale Azure SQL Database and Managed Instance](<02 - Deploy and scale Azure SQL Database and Managed Instance.md>) | DTU/vCore, tiers, serverless, pools, Hyperscale, MI options | ~10 min |
| 3 | [SQL Server on Azure VMs, hybrid and patching](<03 - SQL Server on Azure VMs, hybrid and patching.md>) | VM size, disks, IaaS Agent, hybrid, Update Manager | ~11 min |
| 4 | [Partitioning, compression and sharding](<04 - Partitioning, compression and sharding.md>) | Partition function/scheme, ROW/PAGE, shard keys | ~13 min |
| 5 | [Plan a migration](<05 - Plan a migration.md>) | Stages, online vs offline, tool chooser | ~8 min |
| 6 | [Migrate to Azure SQL Database](<06 - Migrate to Azure SQL Database.md>) | DMS, BACPAC, replication, bcp, ADF | ~8 min |
| 7 | [Migrate to Azure SQL Managed Instance](<07 - Migrate to Azure SQL Managed Instance.md>) | LRS, MI link, Arc migration, copy and move | ~10 min |
| 8 | [Migrate to SQL Server on Azure VMs](<08 - Migrate to SQL Server on Azure VMs.md>) | Backup to URL, distributed AG, other methods | ~6 min |
| 9 | [Troubleshoot a migration](<09 - Troubleshoot a migration.md>) | Common errors and fixes | ~9 min |

## Words you will meet
| Word | Simple meaning |
|------|----------------|
| IaaS | Infrastructure as a Service. Azure gives you a VM; you manage the OS and SQL Server. |
| PaaS | Platform as a Service. Microsoft runs the server, OS and patches; you manage data. |
| vCore | One virtual CPU. You buy compute by vCores. |
| DTU | Database Transaction Unit. A fixed bundle of CPU, memory and disk speed. |
| Elastic pool | Many databases sharing one pot of compute. |
| Hyperscale | SQL Database tier that grows to 128 TB. |
| Azure Arc | Lets Azure manage servers that live outside Azure. |
| Partition | One slice of a table, for example one month of orders. |
| Shard | One of many databases that each hold part of the rows. |
| Online migration | Source keeps working; downtime only at the final switch. |
| Offline migration | Source is stopped for the whole copy. |
| Cutover | The moment apps switch from old database to new one. |
| DMS | Azure Database Migration Service, the main migration engine. |
| LRS | Log Replay Service. Restores your backups into Managed Instance. |
| MI link | Live copy from SQL Server to Managed Instance using a distributed availability group. |

## Exam skills covered
- [ ] Recommend a database offering based on specific requirements → 1
- [ ] Choose an automated deployment method → 2 (short), full detail in Module 4
- [ ] Identify use cases for Azure Arc-enabled SQL services → 1, 3
- [ ] Identify use cases for Azure SQL Database in Microsoft Fabric → 1
- [ ] Plan for table partitioning → 4
- [ ] Recommend a database sharding solution → 4
- [ ] Deploy database offerings on selected platforms → 2, 3
- [ ] Deploy hybrid SQL Server solutions → 3
- [ ] Apply patches and updates for hybrid and IaaS deployments → 3
- [ ] Configure Azure SQL Database for scale and performance → 2
- [ ] Configure Azure SQL Managed Instance for scale and performance → 2
- [ ] Configure SQL Server on Azure Virtual Machines for scale and performance → 3
- [ ] Configure table partitioning → 4
- [ ] Configure data compression → 4
- [ ] Evaluate requirements for a migration → 5
- [ ] Evaluate offline or online migration strategies → 5
- [ ] Implement an online migration strategy → 6, 7, 8
- [ ] Implement an offline migration strategy → 6, 8
- [ ] Implement a migration to Azure → 6, 7, 8
- [ ] Implement a migration between Azure SQL services → 6, 7
- [ ] Implement Azure SQL Managed Instance database copy and move → 7
- [ ] Troubleshoot a migration → 9
