# Verification Log — October 2026

All notes in the six learning-path folders were checked on **25 Oct 2026** against current Microsoft Learn modules and product documentation, ahead of the DP-300 exam on 16 Nov 2026 (skills outline as of 27 Oct 2026).

## How to read this
- **Every fix was made directly in the note.** Each changed or added line ends with **🔄** (in tables, inside the last cell).
- **🔁** in a heading is just a decorative icon (it used to be 🔄 and was swapped so 🔄 only ever means "updated").
- Each section below lists, per folder: the **changes** (old → new, with the Microsoft source), **new files**, files that **needed no change**, and **unverified claims** left as written — review those yourself.
- Where the Microsoft Learn course text and the current product docs disagree, notes follow the **product docs** and mention the course wording (exam questions can lag behind the docs).

## Summary

| Scope | Folders | Changes | New files |
| --- | --- | --- | --- |
| P1a | 1 › Prepare / Deploy IaaS / Deploy PaaS | 43 | 0 |
| P1b | 1 › Migrate to SQL DB / to MI / Migration strategy / to Azure VM | 33 | 2 |
| P2 | 2 › Secure environment | 42 | 0 |
| P3 | 3 › Monitor and optimize resources | 43 | 0 |
| P4 | 4 › Optimize query performance | 31 | 0 |
| P5 | 5 › Automate database tasks | 26 | 0 |
| P6 | 6 › HA/DR | 49 | 0 |
| **Total** | | **267** | **2** |

**New files:** `1 - …/4 - Migrate SQL Server workloads to Azure SQL Database/8 - [NEW] Use Azure Database Migration Service to migrate to Azure SQL Database.md` and `1 - …/5 - Migrate SQL Server workloads to Azure SQL Managed Instance/7 - [NEW] Migrate using Azure Arc.md` (both are new units in the current Learn modules).

**Structural findings:**
- The module **"Prepare to maintain SQL Server-based databases on Azure"** has been removed from Microsoft Learn. Its notes are kept (still exam-relevant).
- **"Migrate SQL Server workloads to Azure SQL"** is now titled **"Design a SQL Server migration strategy"**.
- Units on DMA and the Azure Data Studio migration extension were removed from Learn (tools retired); those notes are kept with retirement callouts.
- Retired or retiring items surfaced: DMA (16 Jul 2025), Azure Data Studio + migration extension (28 Feb 2026), DMS classic SQL scenarios (15 Mar 2026), DEA (15 Dec 2024), Basic Load Balancer (30 Sep 2025), TLS 1.0/1.1 in Azure SQL (31 Aug 2025), Azure Automation Run As accounts (30 Sep 2023), agent-based Hybrid Runbook Worker (31 Aug 2024), Logic Apps ISE (31 Aug 2024), Automated Patching (17 Sep 2027), Intel SGX enclaves (31 Oct 2027), Azure Disk Encryption (15 Sep 2028), Gen4 hardware.

**Reviewer note:** the security check questioned one claim in the new secure-enclaves note ("VBS on any tier except DC-series"). DC-series hardware is the SGX enclave hardware, so the exclusion is plausible, but the current plan doc doesn't spell it out — treat it as unconfirmed.

---
## Verify log: P1a

Scope: `1 - Plan and implement data platform resources/` folders 1 (Prepare to maintain...), 2 (Deploy IaaS...), 3 (Deploy PaaS...).
Checked 25 Oct 2026 against learn.microsoft.com, including the Learn catalog API.

### Module completeness

| Folder | Current Learn module | Result |
| --- | --- | --- |
| 1 - Prepare to maintain SQL Server-based databases on Azure | `prepare-to-maintain-sql-databases-azure`: **no longer exists**. The URL redirects to /training/browse, it's missing from the Learn catalog API, and it's not in the learning path `plan-implement-data-platform-resources`, which now has IaaS, PaaS, migrate-to-SQL-DB and migrate-to-MI. | Files kept. No unit list available to compare. |
| 2 - Deploy IaaS solutions with Azure SQL | https://learn.microsoft.com/en-us/training/modules/deploy-iaas-solutions-with-azure-sql/ (updated 2026-10-21). Units: Intro, IaaS options, Hybrid scenarios, Performance and security, HADR options, Exercise, Assessment, Summary | All substantive units covered (files 1-5). No new file needed. |
| 3 - Deploy PaaS solutions with Azure SQL | https://learn.microsoft.com/en-us/training/modules/deploy-paas-solutions-with-azure-sql/ (updated 2026-10-21). Units: Intro, PaaS options, Single DB, Elastic pool, Hyperscale, Managed instance, Exercise, Assessment, Summary | All substantive units covered (files 1-5). No new file needed. Note: there is no "Introduction" file in this folder, which is fine because intro units are skipped. |

Note: the live Learn units still contain some stale facts, such as Hyperscale "exceed 100 TB / can't revert" and serverless "isn't compatible with geo-replication/LTR". I followed the current product docs, not the training text.

### Changes

| File | Change (old → new) | Source URL |
| --- | --- | --- |
| 1-Prepare/1 - Understand SQL Server in an Azure VM | Added: Marketplace images are now SQL 2016–2025; 2012/2014 images removed | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview?view=azuresql |
| 1-Prepare/1 - Understand SQL Server in an Azure VM | "Premium SSD / Ultra Disk" → "Premium SSD / Premium SSD v2 / Ultra Disk" (+ v2 best price-performance) | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage?view=azuresql |
| 1-Prepare/1 - Understand SQL Server in an Azure VM | Arc: "automated backups, DR, updates" → "automated backups (preview), DR, automatic updates" | https://learn.microsoft.com/en-us/sql/sql-server/azure-arc/backup-local |
| 1-Prepare/1 - Understand SQL Server in an Azure VM | Added Arc bullet: pay-as-you-go licensing + ESU | https://learn.microsoft.com/en-us/sql/sql-server/azure-arc/overview |
| 1-Prepare/2 - Design Azure SQL Database | Hyperscale "automatic scaling" → "automatic storage scaling up to 128 TB; recommended default tier for new OLTP/HTAP" | https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale?view=azuresql |
| 1-Prepare/2 - Design Azure SQL Database | Added: serverless is in GP and Hyperscale; auto-pause is GA only in GP (preview in Hyperscale) | https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql |
| 1-Prepare/2 - Design Azure SQL Database | Added: automatic tuning defaults (FORCE_LAST_GOOD_PLAN on, index create/drop off); MI supports only FORCE_LAST_GOOD_PLAN | https://learn.microsoft.com/en-us/azure/azure-sql/database/automatic-tuning-overview?view=azuresql |
| 1-Prepare/2 - Design Azure SQL Database | Added: SQL database in Fabric GA since Nov 2025 | (established fact in brief) |
| 2-IaaS/2 - IaaS options | "Automated backup & patching" → added "Automated Patching retires 17 Sep 2027 → Azure Update Manager" | (established fact) + https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/sql-server-iaas-agent-extension-automate-management?view=azuresql |
| 2-IaaS/2 - IaaS options | Added: no management modes since Mar 2023; least-privilege; agent installed only when a feature needs it | same IaaS Agent extension URL |
| 2-IaaS/2 - IaaS options | Added: self-installed SQL 2016+ VMs auto-register with the extension | same IaaS Agent extension URL |
| 2-IaaS/2 - IaaS options | PAYG "Pay hourly" → "Pay per second" | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/licensing-model-azure-hybrid-benefit-ahb-change?view=azuresql |
| 2-IaaS/2 - IaaS options | Added: HA/DR license type (free passive replicas); license switching is online | licensing URL above + https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/business-continuity-high-availability-disaster-recovery-hadr-overview?view=azuresql |
| 2-IaaS/2 - IaaS options | Memory optimized "up to 4TB RAM" → "M-series up to ~15 TiB RAM; Ebdsv5 best price-performance" | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-vm-size?view=azuresql |
| 2-IaaS/4 - Performance and security | Ultra Disk max IOPS 160,000 → 400,000 | https://learn.microsoft.com/en-us/azure/virtual-machines/disks-types |
| 2-IaaS/4 - Performance and security | Premium SSD v2 max throughput 1,200 MB/s → 2,000 MB/s | https://learn.microsoft.com/en-us/azure/virtual-machines/disks-types |
| 2-IaaS/4 - Performance and security | Azure Disk Encryption: added "retires 15 Sep 2028; use encryption at host" | https://learn.microsoft.com/en-us/azure/virtual-machines/windows/disk-encryption-overview |
| 2-IaaS/4 - Performance and security | "Azure Security Center" → "Microsoft Defender for Cloud (formerly Azure Security Center)" | Learn unit 4 of the IaaS module |
| 2-IaaS/5 - HADR options | Added: AG listener on Azure VMs: multi-subnet recommended; single subnet → DNN or VNN + Azure Load Balancer | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/availability-group-overview?view=azuresql |
| 2-IaaS/5 - HADR options | FCI storage "Azure shared file storage or S2D" → "Azure shared disks, Premium file shares, or S2D" | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/failover-cluster-instance-overview?view=azuresql |
| 2-IaaS/5 - HADR options | Azure Backup "Azure Backup Vault" → "Recovery Services vault" | https://learn.microsoft.com/en-us/azure/backup/backup-azure-sql-database |
| 3-PaaS/1 - PaaS options | Hyperscale row: storage "Premium Storage" → "Decoupled storage + local SSD cache"; max 100 TB → 128 TB; marked recommended default | https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale?view=azuresql |
| 3-PaaS/1 - PaaS options | Heading "Serverless (Only in General Purpose)" → "(General Purpose and Hyperscale)" | https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql |
| 3-PaaS/1 - PaaS options | Added auto-pause delay range: GP 15 min–7 days; Hyperscale 60 min min. (preview); -1 disables | serverless URL above |
| 3-PaaS/1 - PaaS options | "doesn't support geo-replication, LTR, some background jobs" → these features **prevent auto-pause** (auto-scale still works): geo-rep/FOG, LTR, DNS alias, sync DB, elastic jobs job DB | https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-auto-pause-resume?view=azuresql |
| 3-PaaS/1 - PaaS options | Differential "every 12 hours" → "every 12 or 24 hours (12 h default vCore, 24 h DTU)" | https://learn.microsoft.com/en-us/azure/azure-sql/database/automated-backups-overview?view=azuresql |
| 3-PaaS/1 - PaaS options | Retention "7 to 35 days" → "1 to 35 days (default 7; Basic max 7)" | automated backups URL above |
| 3-PaaS/1 - PaaS options | Added: backup redundancy geo-redundant by default (ZRS/LRS optional); Hyperscale uses snapshots | automated backups URL above |
| 3-PaaS/1 - PaaS options | FOG "Failover is automatic" → Customer managed (recommended) vs Microsoft managed policy | https://learn.microsoft.com/en-us/azure/azure-sql/database/failover-group-sql-db?view=azuresql |
| 3-PaaS/3 - Elastic pool | Added: purchasing models incl. Hyperscale tier; hourly billing for the pool, no per-DB charge | https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-pool-overview?view=azuresql |
| 3-PaaS/3 - Elastic pool | Added: Hyperscale elastic pools GA, 100 TB/pool, up to 4 HA replica pools, adding a DB converts it to Hyperscale | https://learn.microsoft.com/en-us/azure/azure-sql/database/hyperscale-elastic-pool-overview?view=azuresql |
| 3-PaaS/4 - Hyperscale | "beyond 100 TB, no hard maximum" → "10 GB up to 128 TB per DB (100 TB per HS elastic pool)" | https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale?view=azuresql |
| 3-PaaS/4 - Hyperscale | Added: 0–4 HA replicas, up to 30 named replicas, geo-replicas | https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale-replicas?view=azuresql |
| 3-PaaS/4 - Hyperscale | Auth "SQL, Entra, Windows Authentication" → "SQL and Entra; Windows auth for Entra principals is MI-only" | https://learn.microsoft.com/en-us/azure/azure-sql/database/features-comparison?view=azuresql |
| 3-PaaS/4 - Hyperscale | Table max size "100+ TB" → "128 TB" | Hyperscale URL above |
| 3-PaaS/4 - Hyperscale | "can't downgrade" → "reverse migration to General Purpose within 45 days" | Hyperscale URL above (Reverse migrate section) |
| 3-PaaS/4 - Hyperscale | "Pricing slightly higher (storage per TB)" → "simplified pricing Dec 2023, no SQL license fee, no AHB for new HS DBs" | Hyperscale URL above |
| 3-PaaS/5 - Managed instance | DBs per instance "Up to 100" → "Up to 100; 500 on Next-gen GP" | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits?view=azuresql |
| 3-PaaS/5 - Managed instance | Added: Next-gen General Purpose (500 DBs, 32 TB, 3 free IOPS/GB, IOPS/memory sliders) | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/service-tiers-next-gen-general-purpose-use?view=azuresql |
| 3-PaaS/5 - Managed instance | Added: instance pools are GP-only, 2-vCore instances, max 40 instances / 500 DBs per pool | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/instance-pools-overview?view=azuresql |
| 3-PaaS/5 - Managed instance | "In-Memory, FILESTREAM aren't restorable" → FILESTREAM/FileTable never; active In-Memory OLTP not to GP (BC OK) | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/transact-sql-tsql-differences-sql-server?view=azuresql |
| 3-PaaS/5 - Managed instance | Added "Other Current Features": stop/start (GP) | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/instance-stop-start-how-to?view=azuresql |
| 3-PaaS/5 - Managed instance | Added: free offer (1 GP instance/subscription, 12 months, 4 vCores, 720 vCore-h/month, 64 GB, 100/500 DBs) | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/free-offer?view=azuresql |
| 3-PaaS/5 - Managed instance | Added: update policy (SQL 2022 / SQL 2025 default in portal / Always-up-to-date) | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/update-policy?view=azuresql |

Total: 43 changes across 9 files. The new MI section also adds a `---` separator and a heading line; the heading is marked 🔄.

### New files created

None. Both live modules (IaaS, PaaS) are fully covered. The "Prepare to maintain..." module is retired, so there was nothing to compare.

### Files covering units that no longer exist (kept)

- `1 - Prepare to maintain SQL Server-based databases on Azure/1 - Understand SQL Server in an Azure virtual machine.md`
- `1 - Prepare to maintain SQL Server-based databases on Azure/2 - Design Azure SQL Database for cloud-native applications.md`
- `1 - Prepare to maintain SQL Server-based databases on Azure/3 - summary.md`

The whole Learn module `prepare-to-maintain-sql-databases-azure` has been removed from Learn and from the DP-300 learning path. The content is still valid exam background.

### Files checked, no change needed

- 1-Prepare/3 - summary.md
- 1-Prepare/4 - [NEW] Database sharding.md (spot-checked: 128 TB Hyperscale, 30 named replicas: OK)
- 1-Prepare/5 - [NEW] SQL database in Microsoft Fabric.md (spot-checked: GA Nov 2025, MI update-policy note: OK)
- 2-IaaS/1 - Introduction.md
- 2-IaaS/3 - Understand hybrid scenarios.md
- 2-IaaS/6 - [NEW] Table partitioning and data compression.md (spot-checked headings/limits only)
- 2-IaaS/7 - [NEW] Patching hybrid and IaaS SQL Server.md (spot-checked: Automated Patching retirement date, Arc automatic updates Windows-only, MI update-policy defaults: OK)
- 3-PaaS/2 - Explore single SQL database.md (cmdlets/CLI commands match the current Learn unit; "Gen5" is still the portal default, a.k.a. Standard-series (Gen5))

### Unverified / doubtful (left as is)

- 1-Prepare/1: "99.9% uptime guaranteed (when using Premium SSD or Ultra Disk)" for a single VM. This is correct per the older SLA text. The current SLA may also list Premium SSD v2, but I couldn't confirm this on the official SLA page, so I left it.
- 3-PaaS/1 and 3-PaaS/5: log backups "every 5–10 minutes". The current SQL DB doc says "approximately every 10 minutes", and the Learn units still say 5–10. Not changed.
- 2-IaaS/4: Standard HDD max IOPS 2,000. The doc says "2,000, 3,000*", where the higher value applies to some sizes/regions. Left at 2,000.
- 3-PaaS/5: "99.99% SLA uptime" for MI. Not re-checked against the SLA page. Zone-redundant configs may have a higher SLA.
- 2-IaaS/5: AG "up to 9 replicas" (1 primary + 8 secondaries) is consistent with the Learn unit. The 5 synchronous-replica limit isn't mentioned; not added.

---

## Verify log: P1b (migration modules)

Folder prefix: `1 - Plan and implement data platform resources/`. Abbreviations: **F4** = `4 - Migrate SQL Server workloads to Azure SQL Database/`, **F5** = `5 - Migrate SQL Server workloads to Azure SQL Managed Instance/`, **DS** = `Migrate SQL Server workloads to Azure SQL/`, **VM** = `Migrate SQL Server workloads to Azure Virtual Machine/`.

### Module mapping (completeness)

| Folder | Current Learn module | Current unit list vs files |
|---|---|---|
| F4 | [migrate-sql-workloads-azure-sql-databases](https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-sql-databases/) (ms.date 2026-07-23) | Unit 3 is now **"Use Azure Database Migration Service to migrate to Azure SQL Database"** (CLI/PowerShell). The DMA unit was **removed**. New file 8 created. File 3 (ADS extension) and file 4 (DMA) cover units that no longer exist as written. Both are kept. |
| F5 | [migrate-sql-workloads-azure-managed-instances](https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-managed-instances/) | New unit **"Migrate using Azure Arc"** → new file 7. Unit 5 (MI link) gained a **"Choosing between migration methods" (link vs LRS)** table, now summarised in file 4. |
| DS | Module still exists but was **renamed** to **"Design a SQL Server migration strategy"** ([introduction-data-platform-modernization](https://learn.microsoft.com/en-us/training/modules/introduction-data-platform-modernization/)). The old title "Migrate SQL Server workloads to Azure SQL" is now the **learning path** name (DP-3001). | Units 1–4 + summary match. The only other unit is an exercise (cost comparison), which was skipped. |
| VM | [migrate-sql-workloads-azure-virtual-machines](https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-virtual-machines/) (still exists) | Current units: Intro, Review strategies & tools, Distributed AG, Native backup/restore, Compare additional methods. **Removed units:** Azure SQL Migration extension (files 3 and 5) and DMA (file 7). Files kept. Note: **files 3 and 5 are byte-identical duplicates.** |

### Changes

| File | Change (old → new) | Source URL |
|---|---|---|
| F4/1 - introduction.md | Migrate tools "e.g., **DMA**, DMS" → DMS, BACPAC/SqlPackage, transactional replication (DMA retired) | https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-sql-databases/2-choose-right-sql-database-option-azure |
| F4/1 - introduction.md | Assessment "using **Data Migration Assistant**" → Azure Migrate / Azure Arc migration assessment (DMA retired 16 Jul 2025) | https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-sql-databases/1-introduction |
| F4/2 - Choose the right…md | Security row: + "Advanced Threat Protection (part of Microsoft Defender for SQL)" + Microsoft Entra authentication | https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-sql-databases/2-choose-right-sql-database-option-azure |
| F4/2 - Choose the right…md | DMA row "Offline" → marked ❌ retired 16 Jul 2025, removed from the current Learn table | same |
| F4/3 - Use Azure SQL migration extension…md | Schema prep "use .dacpac, SQL Projects, or **DMA**" → .dacpac/SqlPackage, SQL Projects or DMS schema migration (DMA retired) | https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-sql-databases/3-migrate-using-azure-migration-extension |
| F4/3 - Use Azure SQL migration extension…md | + summary bullet: the current unit replaces the extension with DMS via `az datamigration`/PowerShell (link to new file 8) | same |
| F4/5 - Migrate…BACPAC.md | ">200 GB may take longer or fail" → >150 GB: portal/PowerShell import can fail (450 GB temp disk) → use SqlPackage | https://learn.microsoft.com/en-us/azure/azure-sql/database/database-import |
| F4/5 - Migrate…BACPAC.md | + portal imports only a new single DB from Blob (no elastic pool, no firewalled storage). MI: SqlPackage/SSMS only | same |
| F4/5 - Migrate…BACPAC.md | + import with managed identity = preview | same |
| F4/6 - Use an online method…md | "Azure SQL Database supports only SQL Authentication for replication" → Learn says SQL auth. Docs now also allow Microsoft Entra ID (service principal / Arc-enabled SQL Server) | https://learn.microsoft.com/en-us/azure/azure-sql/database/replication-to-sql-database |
| F4/6 - Use an online method…md | "Only Transactional Replication works online (no merge, p2p)" → push subscriber of **transactional or snapshot** only. No merge/p2p/bidirectional/updatable. Publisher = SQL Server 2016+ or SQL MI. + tables need a primary key | same |
| F4/7 - Move data…md | "Moving full schema and data: Azure SQL Migration Extension, BACPAC, or DMA" → Azure DMS, BACPAC/SqlPackage, transactional replication (extension and DMA retired) | https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-sql-databases/2-choose-right-sql-database-option-azure |
| F5/2 - Evaluate migration scenarios.md | "TDE, Advanced Threat Protection, BYOK" → + Vulnerability Assessment. ATP/VA = part of Microsoft Defender for SQL | https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-managed-instances/2-evaluate-migration-scenarios |
| F5/2 - Evaluate migration scenarios.md | Migration-options row "Azure SQL Migration Extension (ADS)" → **SQL Server migration in Azure Arc** (portal, MI link or LRS) | same + https://learn.microsoft.com/en-us/sql/sql-server/azure-arc/migrate-to-azure-sql-managed-instance |
| F5/2 - Evaluate migration scenarios.md | MI link "replicates data bi-directionally" → one-way for 2016–2019, two-way failover+failback with 2022/2025 | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/managed-instance-link-feature-overview |
| F5/3 - Use Log Replay Service…md | + SQL Server 2012+ all editions. DB in restoring state. Only way to restore differential backups on MI | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/log-replay-service-overview |
| F5/3 - Use Log Replay Service…md | + cmdlets `Start/Get/Complete/Stop-AzSqlInstanceDatabaseLogReplay`, `az sql midb log-replay`. Arc can run LRS | same |
| F5/3 - Use Log Replay Service…md | + Business Critical cutover slow (3 replicas seeded) → prefer MI link | same |
| F5/4 - Migrate using MI link.md | "One database → one link" → single-database mode. + multiple-database mode (preview) | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/managed-instance-link-feature-overview |
| F5/4 - Migrate using MI link.md | One-way versions "SQL Server 2016, 2019" → 2016 SP3, **2017 CU31**, 2019 CU20 (+Azure Connect pack 2016/2017). Failover breaks the link | same |
| F5/4 - Migrate using MI link.md | Two-way "SQL Server 2022 … failback offline (online failback in preview)" → **SQL 2022 and SQL 2025**. Online failover **and online failback** via the link. Needs matching update policy (not Always-up-to-date) | same + https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-managed-instances/5-use-link-feature |
| F5/4 - Migrate using MI link.md | + editions Enterprise/Standard/Developer. VNet-local endpoint only | same |
| F5/4 - Migrate using MI link.md | + no failover group with a link. 100 links (GP/BC), 500 (Next-gen GP) | same |
| F5/4 - Migrate using MI link.md | + link vs LRS comparison bullet (from the Learn unit's new table) | https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-managed-instances/5-use-link-feature |
| F5/5 - Move data to SQL MI.md | BACPAC row + "MI can't import BACPAC from portal/PowerShell → SqlPackage/SSMS" | https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-managed-instances/6-synchronize-data |
| DS/4 - Explore the available…md | MI "Up to 8 TB per database" → DB can use whole instance storage: 16 TB (GP/BC premium), 32 TB (Next-gen GP) | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits |
| DS/4 - Explore the available…md | SQL DB "Max size up to 4 TB, scalable through sharding" → 4 TB GP/BC, **Hyperscale 128 TB**, then sharding | https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale |
| VM/2 - Review migrations strategies and tools.md | ADS migration extension row → ❌ retired 28 Feb 2026 → Azure DMS / SSMS migration component | brief (established) + https://learn.microsoft.com/en-us/ssms/migrate/migrate-sql-server-azure-sql |
| VM/2 - Review migrations strategies and tools.md | DMA row → ❌ retired 16 Jul 2025 → Azure Arc / Azure Migrate assessment | brief (established) |
| VM/2 - Review migrations strategies and tools.md | DEA row → ❌ retired 15 Dec 2024 | https://learn.microsoft.com/en-us/sql/dea/database-experimentation-assistant-overview |
| VM/4 -Compare additional migration methods.md | "Windows Import/Export Service" → + now named **Azure Import/Export service** | https://learn.microsoft.com/en-us/azure/import-export/storage-import-export-service |
| VM/6 -Migrate databases using native backup and restore.md | "**Azure Data Explorer**: GUI method" → **Azure Storage Explorer** (Learn text has the wrong product) | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/log-replay-service-overview (lists AzCopy / Azure Storage Explorer for copying backups) |
| VM/8 -Understand distributed AG…md | Add Azure Replica Wizard "classic model, outdated" → classic IaaS VMs retired Sep 2023 → always T-SQL | https://learn.microsoft.com/en-us/azure/virtual-machines/migration-classic-resource-manager-faq |

Total: **33 table rows**, **34 lines** marked ` 🔄` in existing files. Some table-row markers sit inside the last cell, before the closing `|`, so the table still renders.

### New files created

| File | Learn unit covered |
|---|---|
| F4/`8 - [NEW] Use Azure Database Migration Service to migrate to Azure SQL Database.md` | migrate-sql-workloads-azure-sql-databases, unit 3 "Use Azure Database Migration Service to migrate to Azure SQL Database" (`az datamigration` CLI, PowerShell, statuses, performance) |
| F5/`7 - [NEW] Migrate using Azure Arc.md` | migrate-sql-workloads-azure-managed-instances, unit 3 "Migrate using Azure Arc" (+ Arc migration docs: link vs LRS in Arc, prerequisites, limitations) |

### Files that cover units no longer in the current modules (kept, not deleted)

- F4/`3 - Use Azure SQL migration extension…` (the unit slug is now the DMS/CLI unit). Has a retired callout.
- F4/`4 - Explore Data Migration Assistant…` (unit removed). Has a retired callout.
- VM/`3 - Explore Azure SQL Migration extension…` and VM/`5 -Explore Azure SQL Migration extension…` (unit removed; **the two files are identical duplicates**). Both have retired callouts.
- VM/`7 -Understand Data Migration Assistant.md` (unit removed). Has a retired callout.

### Files checked, no change needed

- F4/`4 - Explore Data Migration Assistant…` (retired-tool body is historical only)
- F5/`1 - Introduction.md`
- F5/`6 - [NEW] Managed Instance database copy and move.md` (spot-checked, consistent)
- DS/`1 - Introduction.md`, `2 - Examine data platform modernization.md`, `3 - Plan a data migration.md`, `5 - summary.md`
- DS/`6 - [NEW] Troubleshoot a migration.md` (spot-checked: LRS 30-day limit, one-folder-per-DB/no nested folders, CHECKSUM, single log file, BC seeding delay, MI link FILESTREAM/ADR/Service Broker limits all match current docs)
- VM/`1 - Introduction.md`, `3 - …extension…`, `5 - …extension…`, `7 -Understand Data Migration Assistant.md`

### Unverified or doubtful claims (left as is)

- **VM storage limit inconsistency**: DS/4 says SQL Server on Azure VM "up to **64 TB**". VM/2 says "up to **256 TB**". Both numbers come from current Learn units (different modules). Not verified against VM disk docs.
- VM/2 licensing: "BYOL/AHB … for customers with **Enterprise Agreement & Software Assurance**". Learn says EA only. Actual AHB eligibility is SA or SQL subscription licenses. Not changed.
- VM/1: "FILESTREAM, FileTable, PolyBase, **cross-instance transactions** not yet supported in SQL DB/MI". Matches Learn, but MI now has distributed transactions (MI↔MI, DTC) and data virtualization. Partly outdated, left as is.
- F5/2: MI "backward compatibility to SQL Server 2008, direct migration from SQL 2005" matches the Learn unit. Not verified in product docs.
- F5/5: sync-method "Direction" column (ADF / replication "2-way") isn't stated in Learn.
- F4/6: Learn still says "SQL authentication only" for replication. The docs allow Entra. The note now shows both, so check which one the exam expects.
- Arc migration minimum version: the Learn unit says **SQL Server 2012+**. The Arc docs say migration is available by default **from SQL Server 2014**. Both are noted in new file 7.
- DS/6 [NEW] Troubleshoot: "Arc jobs kept 28 days" and "MI link creation auto-canceled after 6 days" were not re-verified.
- Style note: several original headings already use the 🔄 emoji (e.g. `## 🔄 **Migration Tools & Modes**`). These aren't update markers but could be confused with them.

---

## Verify log: P2 — 2 - Implement a secure environment for a database service

Checked 25 Oct 2026 against learn.microsoft.com. Files read: 22 (20 older + 2 [NEW]). Changes: 42 edits in 17 files (some edits add 1–3 bullets). No git commands were run.

### Completeness (Learn modules vs folders)

| Folder | Learn module | Result |
|---|---|---|
| 1 - Configure database authentication and authorization | training/modules/configure-database-authentication-authorization (8 units) | Units 1, 3, 4, 5, 6 covered by files 1–5. Others are exercise/assessment/summary. No new file needed. |
| 2 - Protect data in-transit and at rest | training/modules/protect-data-transit-rest (10 units) | Units 1–7 covered by files 1–7. File 8 [NEW] covers the "secure enclaves" exam skill. Others are exercise/assessment/summary. No new file needed. |
| 3 - Implement compliance controls for sensitive data | training/modules/implement-compliance-controls-sensitive-data (11 units) | Units 1–8 covered by files 1–7 and 9. File 8 [NEW] (CT/CDC) is extra exam coverage. No new file needed. |

No file covers a unit that no longer exists.

### Changes

| File | Change (old → new) | Source URL |
|---|---|---|
| 1/2 - Describe authentication and identities | SSMS "Universal Authentication" → **Microsoft Entra MFA** option | https://learn.microsoft.com/en-us/training/modules/configure-database-authentication-authorization/3-describe-authentication-identities |
| 1/2 - Describe authentication and identities | Added 2 bullets: Microsoft Entra-only authentication (what it does, CLI/PowerShell, Azure Policy) and who can toggle it (SQL Security Manager yes, SQL Server Contributor no) | https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-only-authentication |
| 1/3 - Describe Security Principals | Logins: added `CREATE LOGIN … FROM EXTERNAL PROVIDER`, GA in SQL DB, MI and SQL Server 2022+ | https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-logins |
| 1/3 - Describe Security Principals | db_backupoperator "(SQL Server only)" → "SQL Server & MI; grants nothing in Azure SQL DB" | https://learn.microsoft.com/en-us/training/modules/configure-database-authentication-authorization/4-describe-security-principals |
| 1/3 - Describe Security Principals | Added: use ##MS_LoginManager## / ##MS_DatabaseManager## instead of loginmanager/dbmanager | https://learn.microsoft.com/en-us/azure/azure-sql/database/security-server-roles |
| 1/3 - Describe Security Principals | Heading "internal Azure-managed roles" → fixed server-level roles (also in SQL Server 2022+) | https://learn.microsoft.com/en-us/azure/azure-sql/database/security-server-roles |
| 1/3 - Describe Security Principals | Added: membership via `ALTER SERVER ROLE` in virtual master; only server admin / Entra admin; Entra groups not supported | https://learn.microsoft.com/en-us/azure/azure-sql/database/security-server-roles · https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-logins |
| 1/3 - Describe Security Principals | "Server-level roles … controlled via Azure Portal/CLI" → 7 fixed roles managed with T-SQL in master | https://learn.microsoft.com/en-us/azure/azure-sql/database/security-server-roles |
| 1/4 - Describe database and object permissions | REFERENCES "See foreign keys" → needed to create a FK referencing the object (Learn wording noted) | https://learn.microsoft.com/en-us/sql/relational-databases/security/permissions-database-engine |
| 1/4 - Describe database and object permissions | "EXECUTE AS … only SQL Server & MI" → EXECUTE AS USER everywhere; EXECUTE AS LOGIN only SQL Server & MI | https://learn.microsoft.com/en-us/training/modules/configure-database-authentication-authorization/5-describe-database-object-permissions |
| 2/2 - Explore TDE | "AES-256 and 3DES" → AES_128/192/256; TRIPLE_DES_3KEY deprecated since 2016 | https://learn.microsoft.com/en-us/sql/t-sql/statements/create-database-encryption-key-transact-sql |
| 2/2 - Explore TDE | Auto key rotation "checks every 60 minutes" → rotates **within 24 hours**; portal/PS/CLI switches | https://learn.microsoft.com/en-us/azure/azure-sql/database/transparent-data-encryption-byok-key-rotation |
| 2/2 - Explore TDE | Added: database-level CMK (SQL DB only, user-assigned MI, DB-level auto-rotation) | https://learn.microsoft.com/en-us/azure/azure-sql/database/transparent-data-encryption-byok-database-level-overview |
| 2/2 - Explore TDE | Added: Azure Disk Encryption retires 15 Sep 2028 → encryption at host | https://learn.microsoft.com/en-us/azure/virtual-machines/disk-encryption-migrate |
| 2/3 - Configure firewall rules | Server-level config "Portal or T-SQL" → Portal, PowerShell, CLI, REST or T-SQL | https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure |
| 2/3 - Configure firewall rules | Removed Azure Data Studio from the "prompted to add IP" sentence (retired 28 Feb 2026) | brief (established fact) |
| 2/3 - Configure firewall rules | "Allow Azure services": added AllowAllWindowsAzureIps 0.0.0.0 rule, includes other customers' subscriptions | https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure |
| 2/3 - Configure firewall rules | Added: 256 server + 256 database rule limits; SQL DB only (MI uses NSG) | https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure |
| 2/3 - Configure firewall rules | Added 3 bullets: Public network access = Disable; network security perimeter (preview; transition → enforced); connection policy Redirect/Proxy/Default + ports | https://learn.microsoft.com/en-us/azure/azure-sql/database/connectivity-settings · https://learn.microsoft.com/en-us/azure/azure-sql/database/network-security-perimeter · https://learn.microsoft.com/en-us/azure/azure-sql/database/connectivity-architecture |
| 2/4 - Object encryption and secure enclaves | "Applications need the CMK and CEK" → driver reads encrypted CEK from DB, decrypts with CMK | https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine |
| 2/5 - Enable encrypted connections | Added: Force Strict Encryption (SQL Server 2022+, TDS 8.0, required for TLS 1.3) | https://learn.microsoft.com/en-us/sql/relational-databases/security/networking/connect-with-strict-encryption |
| 2/5 - Enable encrypted connections | SSMS "Encrypt connection" checkbox → SSMS 20+ Encryption = Mandatory/Strict (+ Trust server certificate) | https://learn.microsoft.com/en-us/sql/relational-databases/security/networking/connect-with-strict-encryption |
| 2/5 - Enable encrypted connections | Added: minimum TLS version; TLS 1.0/1.1 retired 31 Aug 2025; lowest minimum is 1.2; can't revert | https://learn.microsoft.com/en-us/azure/azure-sql/database/connectivity-settings |
| 2/6 - Describe SQL injection | "Advanced Threat Protection" → Microsoft Defender for SQL (Advanced Threat Protection) | https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-sql-introduction |
| 2/6 - Describe SQL injection | "Azure Advanced Threat Protection" (that's a different product, now Defender for Identity) → Microsoft Defender for SQL (ATP) | https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-sql-introduction |
| 2/7 - Understand Azure Key Vault | "HSM-backed encryption" → software-protected (Standard) or HSM-protected (Premium / Managed HSM) | https://learn.microsoft.com/en-us/azure/key-vault/general/overview |
| 2/7 - Understand Azure Key Vault | "access policies (read/write keys)" → Key Vault Crypto Service Encryption User role, or get/list/wrapKey/unwrapKey | https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/setup-steps-for-extensible-key-management-using-the-azure-key-vault |
| 2/7 - Understand Azure Key Vault | Added: managed identity supported (SQL Server 2022 CU17+ on Azure VM / SQL 2025 Arc); on-prem needs service principal | same as above |
| 2/7 - Understand Azure Key Vault | CREATE CREDENTIAL example fixed: IDENTITY = vault name, SECRET = clientId(no hyphens)+secret, FOR CRYPTOGRAPHIC PROVIDER; added connector/EKM-provider prereq comment | same as above |
| 2/7 - Understand Azure Key Vault | CREATE ASYMMETRIC KEY: provider name aligned, added `CREATION_DISPOSITION = OPEN_EXISTING` | same as above |
| 2/7 - Understand Azure Key Vault | "Always Encrypted column encryption" (as an EKM key use) → column-level (cell) encryption; AE CMKs are used by the client driver | same as above |
| 3/2 - Explore data classification | Added: Microsoft Purview Information Protection (MIP) policy mode (tenant Security Admin, no recommendations) | https://learn.microsoft.com/en-us/azure/azure-sql/database/data-discovery-and-classification-overview |
| 3/2 - Explore data classification | Added: Purview label-based access control for Azure SQL DB is retired | same as above |
| 3/3 - Explore server and database audit | "server-level auditing overrides database-level" → server audit always applies; DB audit runs side-by-side | https://learn.microsoft.com/en-us/azure/azure-sql/database/auditing-setup |
| 3/3 - Explore server and database audit | Retention: default 0 = unlimited | https://learn.microsoft.com/en-us/azure/azure-sql/database/auditing-setup |
| 3/3 - Explore server and database audit | Added: managed identity for storage; diagnostic setting for LA/Event Hubs; July 2025 redesign → single `master` folder; Entra failed logins not in audit | https://learn.microsoft.com/en-us/azure/azure-sql/database/auditing-overview |
| 3/4 - Implement DDM | Added datetime() mask row (SQL Server 2022+) | https://learn.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking |
| 3/4 - Implement DDM | Added: granular UNMASK (db/schema/table/column); ALTER ANY MASK; CONTROL always sees data | same as above |
| 3/4 - Implement DDM | `SELECT INTO BackupTable FROM …` (invalid) → `SELECT * INTO BackupTable FROM …` | same as above |
| 3/5 - Implement RLS | Predicate function: added `WITH SCHEMABINDING` (required because security policy SCHEMABINDING defaults to ON) | https://learn.microsoft.com/en-us/sql/relational-databases/security/row-level-security · https://learn.microsoft.com/en-us/sql/t-sql/statements/create-security-policy-transact-sql |
| 3/6 - Defender for SQL | Added SQL Server enabled by Azure Arc + Synapse to protected products | https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-sql-introduction |
| 3/6 - Defender for SQL | VA output "storage account or email" → Express config (no storage, recommended) vs Classic (own storage; classic APIs retire 16 Aug 2027) | https://learn.microsoft.com/en-us/azure/defender-for-cloud/sql-azure-vulnerability-assessment-overview |
| 3/6 - Defender for SQL | Added: subscription-level plans "Defender for Azure SQL Databases" and "Defender for SQL Servers on Machines" | https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-sql-introduction |
| 3/7 - Explore Ledger | Added: available in SQL DB, MI and SQL Server 2022+ | https://learn.microsoft.com/en-us/sql/relational-databases/security/ledger/ledger-overview |
| 3/7 - Explore Ledger | Digest storage: immutable Blob Storage or Azure Confidential Ledger; added verification procs | same as above |
| 3/7 - Explore Ledger | `WITH (LEDGER = ON)` → `WITH (SYSTEM_VERSIONING = ON, LEDGER = ON)` (doc syntax for updatable ledger tables) + append-only syntax | https://learn.microsoft.com/en-us/sql/relational-databases/security/ledger/ledger-how-to-updatable-ledger-tables |
| 3/9 - Implement Microsoft Purview | Lineage: stored-procedure lineage is preview, plus its requirements and limits | https://learn.microsoft.com/en-us/purview/register-scan-azure-sql-database |

### New files created

None. All substantive Learn units are already covered.

### Files checked, no change needed

- 1/1 - Introduction.md
- 1/5 - Identify authentication and authorization failures.md (transient faults, 5 s first retry, exponential backoff, `sys.sql_logins` checks are still correct)
- 2/1 - Introduction.md
- 3/1 - Introduction.md
- 2/8 - [NEW] Always Encrypted with secure enclaves.md (spot-checked: SGX end of support 31 Oct 2027, auto-move to VBS, no attestation for VBS, DC-series max 40 cores / no serverless — all match)
- 3/8 - [NEW] Change tracking and change data capture.md (spot-checked: CDC on any vCore tier and DTU S3+, scheduler 20 s / hourly, db_owner, serverless paused = no CDC — all match)

### Unverified or doubtful claims left as they were

- 1/2: "Entra admin gets sysadmin-like permissions across all databases" — this matches the Learn unit. For Azure SQL Database the docs describe the admin more narrowly (dbo in each user database). Left as Learn wording.
- 2/7: "Subscription admin won't have access to Key Vault unless given permission" — matches the Learn unit, but an Owner can grant themselves data-plane access (RBAC role or access policy). Left as Learn wording.
- 2/7: "Keys and certificates provisioned at VM creation time" (SQL VM Azure Key Vault integration blade) — not re-verified that the portal option still exists in its current form.
- 3/2: "Earlier (pre-SQL Server 2019) classification was stored as extended properties" — plausible (SSMS 17.x), not re-verified.
- 3/6: Exact portal path "Under Microsoft Defender for Cloud, click (Configure)" — the UI wording may differ slightly; not changed.
- 2/8 [NEW]: "VBS available on any tier … except DC-series". The plan doc says VBS is available in "all Azure SQL Database offerings … hardware configuration" and doesn't mention a DC-series exclusion. Not changed (only spot-checked); worth a closer look.
- 3/4: Whether `datetime()` masking is also available in Azure SQL DB/MI. The SQL doc only states SQL Server 2022, so the note says SQL Server 2022+.
- Note: several existing headings already use the 🔄 emoji as a decorative icon (e.g. "## 🔄 **Encryption Types in Always Encrypted**"). Those headings aren't updates.

---

## Verify log — P3: `3 - Monitor and optimize operational resources in Azure SQL/`

Checked 25 Oct 2026. Learn modules (all updated 5 Aug 2026):
- describe-performance-monitoring: 10 units, 7 with content = files 1–7 (+ file 8 [NEW] Intelligent Insights)
- configure-sql-server-resources-optimal-performance: 7 units, 5 with content = files 1–5
- configure-databases-for-optimal-performance: 8 units, 5 with content = files 1–5

Marker convention: ` 🔄` goes at the end of each changed or added line. In table rows it sits inside the last cell, before the closing `|`, so it renders. Table delimiter rows (`| --- |`) can't carry it.

### Changes

| File | Change (old → new) | Source URL |
|---|---|---|
| 0/2 Describe performance monitoring tools | "SQL VM Resource Provider" table row → "SQL IaaS Agent extension (SQL virtual machines resource)"; automated patching now via Azure Update Manager (extension Automated Patching retires 17 Sep 2027) | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/sql-server-iaas-agent-extension-automate-management |
| 0/2 | "install the SQL VM Resource Provider" → "register the VM with the SQL IaaS Agent extension" (Marketplace images auto-registered; self-installed SQL 2016+ auto-registered when detected) | same as above |
| 0/2 | Key takeaway "Register the SQL VM Resource Provider…" → "Use the SQL IaaS Agent extension…" | same as above |
| 0/2 | Added: portal now says "Enable enhanced monitoring", which installs the Azure Monitor agent | https://learn.microsoft.com/en-us/azure/azure-monitor/vm/monitor-vm |
| 0/3 Describe critical performance metrics | Alert scope "single VM / RG / subscription" → list of VMs / one or more RGs / subscription, each "in one Azure region" | https://learn.microsoft.com/en-us/training/modules/describe-performance-monitoring/3-describe-critical-performance-metrics |
| 0/3 | Action-group rows: added Email ARM role, ITSM, secure webhook | same as above |
| 0/5 Explore extended events | Histogram target "Asynchronous" → "Synchronous" (current XE targets doc; the Learn unit still says async) | https://learn.microsoft.com/en-us/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server |
| 0/5 | ETW and Event Pairing rows: marked "SQL Server only – not Azure SQL DB/MI" | same as above |
| 0/5 | Added: SQL DB sessions are database-scoped (`ON DATABASE`), while MI and SQL Server use `ON SERVER`; the ETW example works only on SQL Server | https://learn.microsoft.com/en-us/azure/azure-sql/database/xevent-db-diff-from-svr |
| 0/5 | Added: event_file in SQL DB/MI always writes to Azure Blob Storage (credential: managed identity or SAS); targets available there | same + targets doc |
| 0/6 Describe database watcher (preview) | "Microsoft Fabric Real-Time Analytics (RTA)" → "Real-Time Intelligence (RTI) in Microsoft Fabric (formerly Real-Time Analytics)" (5 lines: features table, data-flow line, tools list, config step, data store table) | https://learn.microsoft.com/en-us/training/modules/describe-performance-monitoring/6-describe-database-watcher |
| 0/6 | Limitation "May not support all Azure SQL features yet" → supports all SQL DB/MI tiers, serverless, Hyperscale, pools and replicas, but not SQL Server on VMs | https://learn.microsoft.com/en-us/azure/azure-sql/database-watcher-overview |
| 0/6 | Added: 100 targets per watcher, 20 watchers per subscription; alerts (added Feb 2025) unavailable with Fabric RTI or a free ADX cluster; watcher and dashboards free, you pay for ADX/Fabric; a new watcher must be started | same as above |
| 0/7 Explore Query Performance Insight | Added: SQL Database only (not MI); requires Query Store (on by default) | https://learn.microsoft.com/en-us/azure/azure-sql/database/query-performance-insight-use |
| 0/7 | Added: top 5–20 queries; docs list Custom metrics as CPU, duration and execution count over 24 h, past week or past month; docs point to database watcher for deeper analysis | same as above |
| 1/2 Optimize Azure storage for SQL VMs (.MD) | Disk-type table: added **Premium SSD v2** row. The Learn unit now lists 5 disk types. | https://learn.microsoft.com/en-us/training/modules/configure-sql-server-resources-optimal-performance/2-explain-how-optimize-azure-storage-virtual-machines |
| 1/2 | "For production use Premium SSD or Ultra Disk" → Premium SSD v2, Premium SSD, or Ultra (Premium SSD v2 with Ebdsv5; Standard HDD/SSD for dev/test only) | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage |
| 1/2 | "Enable Read Caching for data disks" → read-only host caching on Premium SSD only; Premium SSD v2 and Ultra don't support host caching, nor do disks ≥ 4 TiB; stop SQL Server before changing cache settings | https://learn.microsoft.com/en-us/azure/virtual-machines/disks-types + storage best-practices URL |
| 1/2 | "Use Ultra Disk for <1 ms" → Premium SSD v2 or Ultra; on M-series use Write Accelerator for the log drive (never data) | storage best-practices URL |
| 1/2 | Added: format data disks with a 64-KB allocation unit | storage best-practices URL |
| 1/3 Describe VM resizing | Added: constrained vCPU = ½–¼ of parent (M64-32ms / M64-16ms); VM vCore customization (Configurable Constrained Cores, disable SMT) | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-vm-size |
| 1/3 | Added: docs now recommend memory-optimized sizes at 8:1 with ≥4 vCores; Ebdsv5 is the starting point; Mbdsv3/M-series for mission-critical; D/B-series for dev/test | same as above |
| 1/3 | Resize step "1. Stop (restart) the VM" → "Azure restarts the VM as part of a resize" | https://learn.microsoft.com/en-us/training/modules/configure-sql-server-resources-optimal-performance/3-describe-virtual-machine-resizing |
| 1/3 | Takeaway "Start with general-purpose VMs" → dev/test on D/B-series; production on memory-optimized (Ebdsv5) | VM size best-practices URL |
| 1/4 Optimize database storage | Added: Azure SQL DB has only one data file (Hyperscale several) and you can't add files | https://learn.microsoft.com/en-us/training/modules/configure-sql-server-resources-optimal-performance/4-optimize-database-storage |
| 1/4 | **Wrong:** "tempdb files: by default one per vCore, max 16 for Azure SQL DB" → scales with vCores (2 vCores = 4 files), max 16, not configurable via T-SQL | same as above |
| 1/4 | SQL Server tempdb guidance → setup creates one file per logical processor up to 8; add files in multiples of 4 if contention persists | https://learn.microsoft.com/en-us/sql/relational-databases/databases/tempdb-database |
| 1/4 | Added: MI has 12 tempdb data files by default, now **configurable** (max 128; growth and max size; persisted) | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/tempdb-configure |
| 1/4 | Added: memory-optimized tempdb metadata not available in SQL DB/MI | tempdb-database URL |
| 1/5 Control SQL Server resources | **Wrong:** "Cap of CPU % – upper limit when contention happens" → hard cap, enforced even without contention | https://learn.microsoft.com/en-us/training/modules/configure-sql-server-resources-optimal-performance/5-control |
| 1/5 | Added: on MI, RG config replicates to secondaries; SQL Server AG replicas need per-instance config | https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor |
| 1/5 | Added: SQL Server 2025 adds RG to Standard edition plus tempdb space resource governance | same as above |
| 2/2 Explore database maintenance checks | Added: 5%/30% is a rule of thumb; docs say don't rely on fixed thresholds and consider UPDATE STATISTICS; in Azure use REORGANIZE or online + resumable rebuilds | https://learn.microsoft.com/en-us/sql/relational-databases/indexes/reorganize-and-rebuild-indexes |
| 2/2 | Added: **automatic index compaction** (`SET AUTOMATIC_INDEX_COMPACTION = ON`, off by default; SQL DB, MI with Always-up-to-date policy, Fabric) | https://learn.microsoft.com/en-us/sql/relational-databases/indexes/automatic-index-compaction |
| 2/2 | Added: in SQL DB/MI the platform handles backups and integrity checks; index and statistics maintenance stays with you | https://learn.microsoft.com/en-us/training/modules/configure-databases-for-optimal-performance/2-explore-database-maintenance-checks |
| 2/3 Database scoped configuration options | Compat examples "120, 140, or 150" → add 160 and 170 | https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-compatibility-level |
| 2/3 | "Newer compatibility levels (140, 150)" → (140, 150, 160, 170) | same as above |
| 2/3 | Added: default compat for new databases is 170 in SQL DB and SQL Server 2025; MI is 160 (SQL Server 2022 policy) or 170 (2025 / Always-up-to-date policy) | same as above |
| 2/4 Describe automatic tuning | Added: Azure defaults are FORCE_LAST_GOOD_PLAN ON, CREATE_INDEX OFF, DROP_INDEX OFF; configure at server level and use INHERIT | https://learn.microsoft.com/en-us/azure/azure-sql/database/automatic-tuning-overview |
| 2/4 | Added: requires Query Store in READ_WRITE (on by default in SQL Server 2022+ and SQL DB) | https://learn.microsoft.com/en-us/training/modules/configure-databases-for-optimal-performance/4-describe-automatic-tuning |
| 2/4 | Added: DROP_INDEX drops indexes unused for 90 days and duplicates (never unique/PK; on Premium/BC only duplicates); history kept 21 days | automatic-tuning-overview URL |
| 2/5 Describe intelligent query processing | "Many IQP features require compat 150" → features tied to 140/150/160/170; Learn now says use 160 for the full set (Azure SQL DB default is 170) | https://learn.microsoft.com/en-us/training/modules/configure-databases-for-optimal-performance/5-describe-intelligent-query-processing |
| 2/5 | Example `COMPATIBILITY_LEVEL = 150` → `160` (comment `-- 🔄 (was 150)`) | same as above |
| 2/5 | Memory grant feedback: batch mode at 140, row mode at 150, percentile and persistence at 160 | https://learn.microsoft.com/en-us/sql/relational-databases/performance/intelligent-query-processing |
| 2/5 | Added section "6. Newer IQP features (compat 160/170)": PSP, DOP feedback (off by default in SQL Server 2022, `DOP_FEEDBACK = ON`), CE feedback, optimized plan forcing, approximate percentile, OPPO (170), CE feedback for expressions, OPTIMIZED_SP_EXECUTESQL, plus MI update-policy note | IQP URL + https://learn.microsoft.com/en-us/sql/relational-databases/performance/intelligent-query-processing-degree-parallelism-feedback |
| 2/5 | Takeaway "automatic once on compat 150" → 140/150, with 160 for PSP and DOP/CE feedback and 170 for OPPO | IQP URL |

(The 43 scripted edits are grouped into 44 rows here: the Fabric RTI rename row covers 5 edits, and the 3 XE target-row edits are split across two rows.)

### New files created
- None. Every current Learn unit with substantive content is covered by an existing file. The intro, summary, exercise and assessment units were skipped, as the brief says.

### Files covering units that no longer exist
- None. `0/8 - [NEW] Intelligent Insights.md` is an extra topic file, not a Learn unit. It was kept.

### Files checked, no change needed
- `0 - Describe performance monitoring/1 - Introduction.md`
- `0 - Describe performance monitoring/4 - Establish baseline metrics.md` (counters, <20 ms / <10 ms Premium, wait stats all match the current unit)
- `0 - Describe performance monitoring/8 - [NEW] Intelligent Insights.md` (spot-checked: preview, excluded regions, last-hour vs 7-day baseline, 15 patterns, SQLInsights unrelated to retired SQL Insights. All correct.)
- `1 - Configure SQL Server resources for optimal performance/1 - Introduction.md`
- `2 - Configure databases for optimal performance/1 - Introducation.md`

### Unverified or doubtful claims, left as they were
- **QPI Custom tab metrics.** The Learn unit (and the note) say CPU, Data IO, Log IO and memory; the product doc says CPU, duration and execution count. Both are kept, with the doc version added as a note.
- **XE histogram target.** The Learn unit says asynchronous; the current targets doc says synchronous. The note now follows the doc and flags the conflict. On the exam, expect the Learn wording.
- **DOP feedback default.** The doc only says it's *off by default in SQL Server 2022*. Whether it's on by default in Azure SQL DB / SQL Server 2025 wasn't confirmed, so no claim was made.
- **Fragmentation thresholds (5–30% reorganize, >30% rebuild).** Still in the Learn unit, so they were kept. Current SQL docs discourage fixed thresholds; a caveat was added.
- **"Monitoring Insights / Azure Monitor Insights" row (0/2).** The Learn unit still uses this wording; Azure Monitor docs now describe "enhanced monitoring" (VM insights wording seems to be fading). Not renamed.
- **"Last Query Plan Stats – only active at compat 150"** (Learn unit wording; not in the note). Not verified.
- **Automatic plan correction on SQL Server "✅ (2017+)".** Not verified per edition.
- **Azure Monitor metric retention of 93 days.** Matches the Learn unit; not re-checked against Azure Monitor docs.
- **2/1 intro lists "Automatic tuning of some query patterns" as an IQP feature.** Imprecise, since automatic tuning is a separate feature, but harmless. Left.

---

## Verify log: P4 (`4 - Optimize query performance in Azure SQL/`)

Scope: 3 module folders, 16 files, all read in full. Current Learn modules checked (all ms.date 2026-08-05):
- explore-query-performance-optimization (10 units)
- explore-performance-based-design (7 units)
- evaluate-performance-improvements (8 units)

Note: two files already had a `## 🔄 ...` heading emoji before this pass (`2 - Describe normalization.md` line 131 "Denormalization", `4 - Describe dynamic management views and functions.md` line 56 heading). Those are original heading icons, not update markers.

### Changes

Paths are shortened: M1 = `1 - Explore query performance optimization/`, M2 = `2 - Explore performance-based database design/`, M3 = `3 - Evaluate performance improvements/`.

| File | Change (old → new) | Source URL |
|---|---|---|
| M1/3 - Explain estimated and actual query plans.md | Added: `sys.dm_exec_query_plan_stats` is opt-in (`LAST_QUERY_PLAN_STATS` DB-scoped config or TF 2451); SQL 2019+/Azure SQL | https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-plan-stats-transact-sql |
| M1/4 - Describe DMVs and functions.md | Added: SQL Server 2022+ `VIEW SERVER/DATABASE PERFORMANCE STATE`; Azure SQL DB `##MS_ServerStateReader##` | https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql |
| M1/4 - Describe DMVs and functions.md | "SQL Server only: sys.dm_os_wait_stats … not available in Azure SQL Database" → server-scoped. The course calls it N/A in Azure, but docs list it for Azure SQL DB. Use `sys.dm_db_wait_stats` in Azure SQL DB | https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql ; https://learn.microsoft.com/en-us/training/modules/explore-query-performance-optimization/4-describe-dynamic-management-views-functions |
| M1/5 - Explore Query Store.md | "Enabled by default in Azure SQL Database; in SQL Server/Synapse turn it on" → on by default in Azure SQL DB, MI and new SQL Server 2022+ DBs (can't be turned OFF in SQL DB); manual only for 2016–2019/Synapse | https://learn.microsoft.com/en-us/training/modules/explore-query-performance-optimization/5-explore-query-store ; https://learn.microsoft.com/en-us/sql/relational-databases/performance/manage-the-query-store |
| M1/5 - Explore Query Store.md | Added defaults: capture mode AUTO (2019+), max size 1000 MB (2019+), stale threshold 30 d, flush 900 s, interval 60 min, size-based cleanup AUTO, READ_ONLY when full | https://learn.microsoft.com/en-us/sql/relational-databases/performance/manage-the-query-store |
| M1/5 - Explore Query Store.md | Added: Query Store for readable secondaries (preview; always on in SQL DB / MI AUTD; opt-in SQL 2025 `FOR SECONDARY`) | https://learn.microsoft.com/en-us/sql/relational-databases/performance/query-store-for-secondary-replicas |
| M1/6 - Identify problematic query plans.md | "Scalar functions … SQL 2017+ has some improvements" → IQP (compat 140/150); scalar UDF inlining is SQL Server 2019 (compat 150) + Azure SQL | https://learn.microsoft.com/en-us/sql/relational-databases/user-defined-functions/scalar-udf-inlining ; https://learn.microsoft.com/en-us/training/modules/explore-query-performance-optimization/6-identify-problematic |
| M1/6 - Identify problematic query plans.md | Added: Parameter Sensitive Plan optimization (SQL 2022+/Azure SQL, compat 160) | https://learn.microsoft.com/en-us/sql/relational-databases/performance/query-store-hints |
| M1/7 - Describe blocking and locking.md | Added: with ADR (always on in Azure SQL DB/MI), versions go to PVS in the user DB, not TempDB | https://learn.microsoft.com/en-us/sql/relational-databases/accelerated-database-recovery-concepts |
| M1/7 - Describe blocking and locking.md | Added: RCSI ON by default in Azure SQL Database | https://learn.microsoft.com/en-us/sql/relational-databases/performance/optimized-locking |
| M1/7 - Describe blocking and locking.md | Added: optimized locking (TID locking + LAQ; always on in SQL DB and MI AUTD/2025 policy; opt-in SQL Server 2025; needs ADR, LAQ needs RCSI) | https://learn.microsoft.com/en-us/sql/relational-databases/performance/optimized-locking |
| M1/7 - Describe blocking and locking.md | Isolation table "Read Committed (default)" → "default in SQL Server; Azure SQL DB defaults to RCSI" | https://learn.microsoft.com/en-us/sql/relational-databases/performance/optimized-locking |
| M1/7 - Describe blocking and locking.md | Snapshot row "Locks held until end: Yes (versioned)" → "No (reads use row versions; transaction-level consistency, update conflicts possible)" | https://learn.microsoft.com/en-us/training/modules/explore-query-performance-optimization/7-describe-blocking-locking |
| M2/3 - Choose appropriate data types.md | Implicit conversion: "SQL converts 12345 to text" → INT has higher precedence, so the VARCHAR column is converted to INT on every row (blocks seek, may fail) | https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-type-precedence-transact-sql |
| M2/3 - Choose appropriate data types.md | BIT "Only 1 bit of storage" → "Up to 8 BIT columns share 1 byte" | https://learn.microsoft.com/en-us/sql/t-sql/data-types/bit-transact-sql (from knowledge of the bit doc; not fetched this session) |
| M2/3 - Choose appropriate data types.md | Added rows: native `JSON` type (GA SQL DB, MI 2025/AUTD policy, SQL 2025) and `VECTOR(n)` (float32 up to 1,998 dims, float16 option; SQL DB, MI AUTD, SQL 2025) | https://learn.microsoft.com/en-us/sql/t-sql/data-types/json-data-type ; https://learn.microsoft.com/en-us/sql/t-sql/data-types/vector-data-type |
| M2/4 - Design indexes.md | Added: ordered columnstore (`ORDER (...)`, SQL 2022+, SQL DB, MI) | https://learn.microsoft.com/en-us/sql/relational-databases/indexes/ordered-columnstore-indexes |
| M2/4 - Design indexes.md | Added: `OPTIMIZE_FOR_SEQUENTIAL_KEY` for last-page insert (PAGELATCH_EX) contention, SQL 2019+/Azure SQL, default OFF | https://learn.microsoft.com/en-us/sql/t-sql/statements/create-index-transact-sql ; https://learn.microsoft.com/en-us/troubleshoot/sql/database-engine/performance/resolve-pagelatch-ex-contention |
| M2/4 - Design indexes.md | Added: rowgroup max 1,048,576 rows; tuple mover compresses CLOSED delta rowgroups (~5 min); `REORGANIZE WITH (COMPRESS_ALL_ROW_GROUPS = ON)` | https://learn.microsoft.com/en-us/sql/relational-databases/indexes/columnstore-indexes-data-loading-guidance |
| M3/2 - Describe wait statistics.md | Added: `sp_configure` MAXDOP not for Azure SQL DB → `ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP`; new SQL DBs default MAXDOP 8 | https://learn.microsoft.com/en-us/azure/azure-sql/database/configure-max-degree-of-parallelism |
| M3/2 - Describe wait statistics.md | Added: CXCONSUMER split (SQL 2016 SP2 / 2017 CU3) | https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql |
| M3/2 - Describe wait statistics.md | "PAGEIOLATCH_UP (on TempDB)" → **PAGELATCH_UP** (in-memory PFS/GAM/SGAM latch, page 2:1:1, no disk I/O) | https://learn.microsoft.com/en-us/training/modules/evaluate-performance-improvements/2-describe-wait-statistics ; https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql |
| M3/3 - Tune and maintain indexes.md | "Resumable Indexes (SQL Server 2019+)" → resumable rebuild 2017+, resumable create 2019+, Azure SQL DB & MI | https://learn.microsoft.com/en-us/sql/relational-databases/indexes/guidelines-for-online-index-operations |
| M3/3 - Tune and maintain indexes.md | Added: RESUME / ABORT / MAX_DURATION (minutes); survives failover; ELEVATE_RESUMABLE / ELEVATE_ONLINE | https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-index-transact-sql ; https://learn.microsoft.com/en-us/sql/relational-databases/indexes/guidelines-for-online-index-operations |
| M3/4 - Understand query hints.md | Fixed broken table row: `{LOOP \| MERGE \| HASH} JOIN` (unescaped pipes had split it into 5 cells) | https://learn.microsoft.com/en-us/training/modules/evaluate-performance-improvements/4-understand-query-hints |
| M3/4 - Understand query hints.md | Added: `sys.query_store_query_hints`, `sp_query_store_clear_hints`; QS hints persist and override statement hints/plan guides | https://learn.microsoft.com/en-us/sql/relational-databases/performance/query-store-hints |
| M3/4 - Understand query hints.md | Added: SQL 2025/Azure SQL `@replica_group_id` for secondary-replica hints | https://learn.microsoft.com/en-us/sql/relational-databases/performance/query-store-hints |
| M3/5 - Explore performance scenarios.md | "These DMVs update about every 15 seconds" → dm_db_resource_stats every 15 s (~1 h kept); server_resource_stats rows per 15 s but aggregated in 5–10 min windows (~14 days) | https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database ; https://learn.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-server-resource-stats-azure-sql-database |
| M3/5 - Explore performance scenarios.md | `INSTANCE_LOG_GOVERNOR` → `INSTANCE_LOG_RATE_GOVERNOR` (course still writes the short name) | https://learn.microsoft.com/en-us/azure/azure-sql/database/resource-limits-logical-server ; https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql |
| M3/5 - Explore performance scenarios.md | `HADR_THROTTLE_LOG_RATE_SEND_RECV` → `HADR_THROTTLE_LOG_RATE_SEND_RECV_QUEUE_SIZE` (2 tables); added `HADR_THROTTLE_LOG_RATE_MISMATCHED_SLO` | https://learn.microsoft.com/en-us/azure/azure-sql/database/resource-limits-logical-server |
| M3/5 - Explore performance scenarios.md | "limits background worker threads" → concurrent workers by tier/compute size; error → error 10928 | https://learn.microsoft.com/en-us/azure/azure-sql/database/resource-limits-logical-server |

Total: 31 logical changes (37 marked lines) across 11 files.

### New files created

None. Every current unit with substantive content (non-intro/summary/exercise/assessment) is covered by an existing file. No existing file covers a removed unit.

### Files checked, no change needed

- M1/1 - Introduction.md
- M1/2 - Understand query plans.md
- M2/1 - Introduction.md
- M2/2 - Describe normalization.md
- M3/1 - Introduction.md

### Unverified or doubtful claims (left as is)

- M1/2: "Third-party tools like SentryOne and ApexSQL". SentryOne Plan Explorer is now sold under the SolarWinds brand (not Microsoft docs, so left unchanged).
- M3/2: `DBCC SQLPERF('sys.dm_os_wait_stats', CLEAR)`. The syntax is correct for SQL Server/MI. I couldn't confirm whether Azure SQL Database supports clearing it.
- M2/4: load-time demo numbers (8 s columnstore vs 20 s rowstore for ~1M rows). These come from the course demo and still match the Learn unit, but they are not a general rule.
- M2/3: the BIT storage fix comes from the bit data-type doc as I know it. I didn't fetch that page this session.
- M3/5: `sys.dm_instance_resource_governance` (MI). The Learn unit lists `sys.dm_user_db_resource_governance` twice (a typo), and the note's MI DMV name wasn't fetched. I believe it's correct.

---

## Verify log: P5 — `5 - Automate database tasks for Azure SQL/`

Learn path checked: https://learn.microsoft.com/en-us/training/paths/automate-tasks-sql-server/ (updated 2026-10-21)
- Module 1 `configure-automatic-deployment-azure-sql-database`: 8 units (intro, deployment models, ARM/Bicep, PowerShell, CLI, exercise, assessment, summary). All substantive units covered by files 2–5.
- Module 2 `schedule-tasks-using-sql-server-agent`: 6 units (intro, maintenance plan, task status notifications, knowledge check, exercise CPU alert, summary). All covered by files 1–3.
- Module 3 `manage-azure-paas-resources-using-automated-methods`: 9 units (intro, elastic jobs, Azure Automation, runbook, Logic Apps, monitor, exercise, assessment, summary). All covered by files 1–6.
- No file covers a unit that no longer exists. The [NEW] files (M1 #6, M2 #4) cover exam-outline skills that have no Learn unit.
- Gap: the exam outline (27 Oct 2026) skill "Troubleshoot automated database tasks" has no Learn unit. No new file was created (the brief only allows new files for Learn units). Troubleshooting bullets were added to M3 #6 instead.

### Changes

| File | Change (old → new) | Source URL |
| --- | --- | --- |
| 1/2 - Describe deployment models in Azure.md | "Some Azure SQL and PostgreSQL commands are CLI-exclusive" → "Some commands for Azure Database for PostgreSQL and MySQL are CLI-exclusive" | https://learn.microsoft.com/en-us/training/modules/configure-automatic-deployment-azure-sql-database/2-describe-deployment-models-azure |
| 1/2 - Describe deployment models in Azure.md | Added: Bicep is Microsoft's recommended authoring language and compiles to ARM JSON | same as above |
| 1/2 - Describe deployment models in Azure.md | Added: `bicep decompile` converts an exported ARM template to Bicep | same as above |
| 1/3 - ARM templates and Bicep.md | Added: Azure PowerShell needs the Bicep CLI installed manually; neither PowerShell nor the CLI can deploy a remote .bicep file | https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-powershell ; https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-cli |
| 1/4 - Automate deployment by using PowerShell.md | `-ComputeGeneration Gen4` (×2) → `Gen5`, with a comment that Gen4 is retired (Gen5 / G8IM / G8IH are the valid choices) | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/service-tiers-managed-instance-vcore ; https://azure.microsoft.com/updates/support-has-ended-for-gen-4-hardware-on-azure-sql-database |
| 1/5 - Automate deployment by using Azure CLI.md | Added: local .bicep / .bicepparam files deploy directly with `az deployment group create` | https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-cli |
| 2/1 - Introduction.md | Added: on MI, SQL Agent is always running and its settings are read-only; Azure SQL Database has no Agent (use elastic jobs) | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/job-automation-managed-instance |
| 2/3 - Describe task status notifications.md | Added: MI Database Mail profile must be named `AzureManagedInstance_dbmail_profile`, and the NSG needs an outbound SMTP rule | same as above |
| 2/3 - Describe task status notifications.md | Added: SQL Agent alerts, Pager and NetSend aren't supported on MI; use Azure Monitor alerts | same as above |
| 3/2 - Explore Elastic jobs.md | Added: elastic jobs GA since April 2024; work with all tiers | https://azure.microsoft.com/en-us/updates/general-availability-azure-sql-updates-for-midapril-2024/ ; https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-overview |
| 3/2 - Explore Elastic jobs.md | Added component row: job agent tiers JA100 (default) / JA200 / JA400 / JA800 | https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-overview |
| 3/2 - Explore Elastic jobs.md | Added: recommended auth is Entra ID with a user-assigned MI (system-assigned MI not supported); database-scoped credentials are the alternative; one method per agent | same as above |
| 3/2 - Explore Elastic jobs.md | Added: elastic job private endpoints (service-managed, approved on each target server) | same as above |
| 3/2 - Explore Elastic jobs.md | Added: T-SQL alternative `jobs.sp_add_target_group` / `jobs.sp_add_target_group_member` | same as above |
| 3/2 - Explore Elastic jobs.md | Added: Hyperscale and auto-pause serverless databases aren't supported as the job database | same as above |
| 3/2 - Explore Elastic jobs.md | Comparison table "Administration: Azure Portal, CLI, PowerShell" → "Azure Portal, PowerShell, T-SQL, REST API (no `az sql` elastic-job commands)" | https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-overview ; https://learn.microsoft.com/en-us/cli/azure/sql |
| 3/3 - Understand Azure Automation.md | Added component row: managed identity (`Connect-AzAccount -Identity`); Run As accounts retired 30 Sep 2023 | https://learn.microsoft.com/en-us/azure/automation/migrate-run-as-accounts-managed-identity |
| 3/3 - Understand Azure Automation.md | Added: Hybrid Runbook Worker is extension-based only; the agent-based (Log Analytics agent) worker retired 31 Aug 2024 | https://learn.microsoft.com/en-us/azure/automation/automation-hybrid-runbook-worker |
| 3/4 - Build an automation runbook.md | "Runtime: PowerShell 7.2 recommended" → "PowerShell 7.4 (or 7.6) recommended; 7.1/7.2 out of support; 5.1 still available; Python 3.10" | https://learn.microsoft.com/en-us/azure/automation/automation-runbook-types |
| 3/4 - Build an automation runbook.md | Added: `Invoke-Sqlcmd` needs the SqlServer module; PS 7.4+ runtime environments include a default Az version | same as above |
| 3/4 - Build an automation runbook.md | Added a script comment: the FQDN is built from the DB name, which works only because the server and DB are both named GSData (sample copied verbatim from Learn) | https://learn.microsoft.com/en-us/training/modules/manage-azure-paas-resources-using-automated-methods/4-build-automation-runbook |
| 3/5 - Logic Apps.md | ISE row → struck through: "Retired 31 Aug 2024, migrate to Standard" | https://learn.microsoft.com/en-us/azure/logic-apps/connect-virtual-network-vnet-isolated-environment-overview ; https://azure.microsoft.com/updates/integration-services-environment-will-be-retired-on-31-august-2024-transition-to-logic-apps-standard |
| 3/5 - Logic Apps.md | "Azure Active Directory / AAD" → Microsoft Entra ID; auth types updated (SQL auth, Entra integrated, Logic Apps managed identity [recommended], service principal, connection string for Standard built-in) | https://learn.microsoft.com/en-us/azure/connectors/connectors-create-api-sqlazure |
| 3/5 - Logic Apps.md | Added: Standard built-in SQL triggers need change tracking on the table | same as above |
| 3/5 - Logic Apps.md | Added: MI without a data gateway → public endpoint on port 3342 | same as above |
| 3/6 - Monitor automated tasks.md | Added "Alerts for Elastic Jobs" section: Azure Monitor alerts with the **Failed Elastic jobs** metric plus an action group; history purged after 45 days (`jobs.sp_purge_jobhistory`); `jobs_reader` role; troubleshooting cmdlets / runbook job streams | https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-overview ; https://learn.microsoft.com/en-us/training/modules/manage-azure-paas-resources-using-automated-methods/6-monitor-automated-tasks |

### New files created
- None. Every current Learn unit with substantive content is already covered.

### Checked, no change needed
- 1/1 - Introduction.md
- 1/6 - [NEW] Monitor and troubleshoot deployments.md (spot-checked: 800 / 700→600 history limits, CanNotDelete lock, and "what-if/validate trigger deletion" all match https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-history-deletions)
- 2/2 - Create a SQL Server maintenance plan.md
- 2/4 - [NEW] Troubleshoot SQL Server Agent jobs.md (spot-checked: MI history fixed at 1,000/100, MI no alerts/proxies, mail profile name, elastic history 45 days; all match the docs)
- 3/1 - Introduction.md
- 3/6 Activity log / Log Analytics / Total Jobs alert content: matches the current Learn unit. Log Analytics agent (MMA) retirement doesn't affect Automation diagnostic settings; it only affected the agent-based Hybrid Worker, which is now noted in 3/3.
- No AzureRM usage was found anywhere in the folder; all cmdlets are Az.*.

### Unverified / doubtful (left as is)
- The Learn unit "Explore Elastic jobs" says target groups can be created with "PowerShell, T-SQL, or Azure CLI". The Azure CLI `az sql` reference has no elastic-job command group, so the CLI part looks wrong. I fixed the note's comparison table but did not repeat this claim anywhere.
- 2/4 [NEW] file, "Docs conflict on CmdExec": the MI job-automation doc lists "OS command/PowerShell steps" as a supported job step type, but its limitations list says "PowerShell Core isn't supported" and "scripts on disk aren't supported". Left as written (the file already flags the conflict).
- 2/2: "The Maintenance Plan Wizard creates a separate schedule for each plan", and whether maintenance plans are supported on MI. Not verified.
- 1/3: "Azure CLI installs the Bicep CLI automatically" (the parenthetical I added) is consistent with Bicep install docs but was not quoted from a fetched page. The deploy-powershell page confirms only the PowerShell half.
- 2/1: "SQL Server Agent is a Windows service". It also runs on SQL Server on Linux. Not changed.

---

## Verify log: P6 (Plan and implement a high availability and disaster recovery environment)

Folder: `6 - Plan and implement a high availability and disaster recovery environment/` (3 modules, 18 files, all read in full).
Abbreviations: M1 = `1 - Describe high availability and disaster recovery strategies`, M2 = `2 - Explore IaaS and PaaS solutions for high availability and disaster recovery`, M3 = `3 - Back up and restore databases`.

Note: `M3/2 - Back up and restore SQL Server running on Azure virtual machines.md` already had a 🔄 in the heading `## 🔄 **Types of SQL Server Backups**`. That emoji is part of the original heading style. It isn't an update marker.

### Changes

| File | Change (old → new) | Source URL |
| --- | --- | --- |
| M1/3 - Explore HA and DR options | FCI "needs ... an internal load balancer (ILB)" → multi-subnet (recommended, no LB), or DNN, or VNN + Standard LB in a single subnet | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/failover-cluster-instance-overview |
| M1/3 | FCI storage "Azure Shared Disks or Premium File Share" → shared disks, Premium file shares, S2D, Azure Elastic SAN (SQL 2022+) | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/failover-cluster-instance-overview |
| M1/3 | "In Azure, you need an ILB ... in FCI setups" → DNN (recommended) or VNN + Standard LB only for a single subnet; multi-subnet needs neither; Basic LB retired 30 Sep 2025 | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/hadr-windows-server-failover-cluster-overview ; https://azure.microsoft.com/updates/azure-basic-load-balancer-will-be-retired-on-30-september-2025-upgrade-to-standard-load-balancer |
| M1/4 - Azure HA/DR features for VMs | Zone latency "usually < 1 ms" → Microsoft targets round-trip < ~2 ms | https://learn.microsoft.com/en-us/azure/reliability/availability-zones-overview |
| M1/5 - HA/DR for PaaS | Active geo-replication "Create read/write replicas" → up to 4 readable (read-only) geo-secondaries | https://learn.microsoft.com/en-us/azure/azure-sql/database/active-geo-replication-overview |
| M1/5 | "99.99% SLA" → 99.99% (99.995% for zone-redundant Business Critical/Premium) | https://azure.microsoft.com/en-us/blog/understanding-and-leveraging-azure-sql-database-sla/ ; https://learn.microsoft.com/en-us/azure/azure-sql/database/high-availability-sla-local-zone-redundancy |
| M1/5 | Added: zone redundancy for SQL DB GP/BC/Hyperscale/Premium (not Basic/Standard) and SQL MI GP/BC | https://learn.microsoft.com/en-us/azure/azure-sql/database/high-availability-sla-local-zone-redundancy ; https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/high-availability-sla-local-zone-redundancy |
| M1/6 - HA/DR solution for IaaS | Added: SQL Server 2022+ contained AGs replicate logins/permissions/Agent jobs (own master/msdb) | https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/contained-availability-groups-overview |
| M1/7 - Hybrid solutions | "PaaS doesn't support hybrid HADR" → Managed Instance link gives hybrid DR between SQL Server and SQL MI | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/managed-instance-link-feature-overview |
| M1/7 | Transactional replication: "on-prem → MI subscriber only; can't replicate from Azure back to on-premises" → SQL DB can also be a push subscriber; MI can be Publisher/Distributor and push to SQL Server | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/replication-transactional-overview |
| M1/7 | Added: MI link exception (SQL 2016–2019 one-way; SQL 2022/2025 two-way with failback; license-free DR replica) | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/managed-instance-link-feature-overview |
| M1/7 | Summary "except for replication from on-prem to Azure MI" → transactional replication and MI link | same as above |
| M2/1 - Introduction | "Auto-failover groups" → "Failover groups (formerly auto-failover groups)" | https://learn.microsoft.com/en-us/azure/azure-sql/database/failover-group-sql-db |
| M2/2 - Failover clusters | Witness "Use Cloud Witness instead of a disk or file share" → cloud witness whenever possible; disk witness for FCI with Azure shared disks; file share as fallback | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/hadr-windows-server-failover-cluster-overview |
| M2/2 | "From Windows Server 2019 onward, DNN replaces VNN for FCI" → DNN for FCI needs SQL Server 2019 CU2+ on Windows Server 2016+; multi-subnet needs neither LB nor DNN | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/hadr-windows-server-failover-cluster-overview |
| M2/3 - Configure AGs | ILB "Required for AG listener in Azure" → only for VNN listener in a single subnet; alternatives multi-subnet or DNN listener (2016 SP3 / 2017 CU25 / 2019 CU8+) | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/availability-group-overview |
| M2/3 | Added: use Standard LB (Basic retired 30 Sep 2025) with Floating IP | https://azure.microsoft.com/updates/azure-basic-load-balancer-will-be-retired-on-30-september-2025-upgrade-to-standard-load-balancer ; https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/availability-group-overview |
| M2/3 | "AG spans multiple subnets or regions, each needs a separate ILB" → each region needs an LB with VNN; multi-subnet needs none | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/availability-group-overview |
| M2/3 | "Volume Shadow Service" → "Volume Shadow Copy Service" | https://learn.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service |
| M2/4 - Active geo-replication | "All replicas must use the same service tier & compute size" → same service tier required; same compute size/backup redundancy strongly recommended | https://learn.microsoft.com/en-us/azure/azure-sql/database/active-geo-replication-overview |
| M2/4 | Added rows: up to 4 geo-secondaries (chaining for more); Hyperscale supported; standby (license-free) DR replica | https://learn.microsoft.com/en-us/azure/azure-sql/database/active-geo-replication-overview |
| M2/4 | Cross-subscription "requires programmatic setup (CLI/PowerShell/REST)" → portal works in same Entra tenant; different tenant = SQL auth + T-SQL (or REST); not with Entra-only auth | https://learn.microsoft.com/en-us/azure/azure-sql/database/active-geo-replication-overview |
| M2/5 - Failover groups | "Auto-failover groups are a built-in HA and DR solution" → "Failover groups (formerly auto-failover groups) ... cross-region DR" | https://learn.microsoft.com/en-us/azure/azure-sql/database/failover-group-sql-db |
| M2/5 | Scope "multiple databases in one group" → SQL DB: some/all DBs, multiple groups per server; SQL MI: all user DBs, one group per instance, system DBs not replicated | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/failover-group-sql-mi |
| M2/5 | Policy table "Automatic: Enabled (default)" → Customer managed (`Manual`, recommended) vs Microsoft managed (`Automatic`, all groups in region, min 1 h grace) | https://learn.microsoft.com/en-us/azure/azure-sql/database/failover-group-sql-db |
| M2/5 | Read-only "listener disabled during failover / can stay enabled" → read-only failover to the primary is disabled by default; can be enabled | https://learn.microsoft.com/en-us/azure/azure-sql/database/failover-group-sql-db |
| M2/5 | GracePeriodWithDataLossHours "how long Azure waits for the secondary to sync" → how long Azure waits before a forced (Microsoft-managed) failover; default and minimum 1 h | https://learn.microsoft.com/en-us/azure/azure-sql/database/failover-group-sql-db |
| M2/5 | Comparison: FOG automatic failover "Yes" → only with Microsoft-managed policy | https://learn.microsoft.com/en-us/azure/azure-sql/database/failover-group-sql-db |
| M2/5 | Comparison: multiple secondaries "Geo Yes / FOG No" → Geo up to 4; FOG one secondary (multiple secondaries preview for SQL DB) | https://learn.microsoft.com/en-us/azure/azure-sql/database/failover-group-sql-db |
| M2/5 | Setup step "auto-failover group" → "failover group" | https://learn.microsoft.com/en-us/azure/azure-sql/database/failover-group-sql-db |
| M2/5 | Added: license-free standby DR replica; MI requirements (same DNS zone / DNSZonePartner, global VNet peering) | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/failover-group-sql-mi ; https://learn.microsoft.com/en-us/azure/azure-sql/database/active-geo-replication-overview |
| M2/6 - Monitor availability | "auto-failover groups" → "failover groups" | https://learn.microsoft.com/en-us/azure/azure-sql/database/failover-group-sql-db |
| M2/6 | "Use XEvents on MI to track backup history" → XEvents or `msdb.dbo.backupset` (MI), `sys.dm_database_backups` (SQL DB) | https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-database-backups-azure-sql-database ; https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/automated-backups-overview |
| M3/2 - Backup/restore SQL Server on VMs | Log backup "shrinks the transaction log file" → truncates the log (marks space reusable; doesn't shrink the file) | https://learn.microsoft.com/en-us/sql/relational-databases/logs/the-transaction-log-sql-server |
| M3/3 - Back up a SQL Server VM | Added section 1b: Azure Backup for SQL Server in Azure VMs (workload extension, full/diff/log to vault, 15-min log, PITR per DB; snapshot backup preview) | https://learn.microsoft.com/en-us/azure/backup/backup-azure-sql-database |
| M3/3 | Added: block blob ~195 GB per blob, stripe up to 64 URLs (~12.8 TB), page blob 1 TB | https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/sql-server-backup-to-url |
| M3/3 | Added: SQL Server 2022+ backup to S3-compatible object storage (`s3://`, `S3 Access Key` credential) | https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/sql-server-backup-to-url-s3-compatible-object-storage |
| M3/3 | "SQL Server IaaS Agent Extension, also called the SQL Server resource provider" → Automated Backup via SQL IaaS Agent extension (SQL virtual machines resource), configures Managed Backup to Azure | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/automated-backup |
| M3/3 | Added: Automated Backup retention 1–90 days (default 90), log every 5–60 min, SQL 2016+ Std/Ent/Dev, full recovery, default or single named instance | https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/automated-backup |
| M3/3 | Takeaways table: added row "Azure Backup for SQL Server in Azure VM" | https://learn.microsoft.com/en-us/azure/backup/backup-azure-sql-database |
| M3/4 - Backup/restore SQL DB and MI | "Differential every 12 hours" → every 12 or 24 hours (vCore default 12 h, DTU default 24 h) | https://learn.microsoft.com/en-us/azure/azure-sql/database/automated-backups-overview |
| M3/4 | "Log backup every 5–10 minutes" → approximately every 10 minutes | https://learn.microsoft.com/en-us/azure/azure-sql/database/automated-backups-overview ; https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/automated-backups-overview |
| M3/4 | Added: Hyperscale uses storage snapshots (no full/diff/log) | https://learn.microsoft.com/en-us/azure/azure-sql/database/automated-backups-overview |
| M3/4 | "Backups stored in RA-GRS" → default GRS; options LRS/ZRS/GRS/GZRS; geo-restore only with GRS/GZRS; Hyperscale redundancy set at creation only | https://learn.microsoft.com/en-us/azure/azure-sql/database/automated-backups-overview |
| M3/4 | PITR "7 days, extendable up to 35" → 1–35 days, default 7; Basic 1–7 days | https://learn.microsoft.com/en-us/azure/azure-sql/database/automated-backups-overview |
| M3/4 | "Logical server deleted → all backups lost" → DBs and PITR backups lost, LTR backups kept; deleted MI: contact support within 5 days | https://learn.microsoft.com/en-us/azure/azure-sql/database/automated-backups-overview ; https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/automated-backups-overview |
| M3/4 | MI copy-only reason "Azure is managing the transaction log" → Azure manages the backup chain; added: service-managed TDE DBs can't be COPY_ONLY backed up | https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/copy-only-backups-sql-server |
| M3/4 | "Cannot use PaaS backups to restore to on-prem SQL Server, and vice versa" → SQL DB can't; MI copy-only backups can restore to SQL Server 2022/2025 with matching update policy; SQL Server backups can restore to MI | https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/restore-database-to-sql-server |
| M3/4 | LTR line: added weekly/monthly/yearly full backups up to 10 years | https://learn.microsoft.com/en-us/azure/azure-sql/database/automated-backups-overview |

Total: 49 change rows, 63 lines marked 🔄 across 13 files.

### New files created

None. All three current Learn modules match the folders unit for unit (checked 25 Oct 2026):
- https://learn.microsoft.com/en-us/training/modules/describe-high-availability-disaster-recovery-strategies/ (units 1–7 covered; 8 assessment, 9 summary skipped)
- https://learn.microsoft.com/en-us/training/modules/explore-iaas-paas-platform-tools-for-high-availability-disaster-recovery/ (units 1–6 covered; exercise, assessment, summary skipped). `7 - [NEW] Test and troubleshoot HA-DR solutions.md` is an extra file with no matching Learn unit (it covers an exam skill). Kept.
- https://learn.microsoft.com/en-us/training/modules/backup-restore-databases/ (units 1–4 covered; exercise "Backup to URL", assessment, summary skipped)

No file covers a unit that no longer exists.

### Files checked, no change needed

- M1/1 - Introduction.md
- M1/2 - Describe recovery time objective and recovery point objective.md
- M2/7 - [NEW] Test and troubleshoot HA-DR solutions.md (spot-checked: 1 failover per 15 min per DB/pool/MI; Hyperscale readable secondaries have no failover command; seeding up to 500 GB/h SQL DB and 360 GB/h MI; MI FOG creation cancelled after 6 days; customer-managed policy recommended; grace period min 1 h; listener names and 30 s TTL; geo-secondary PITR retention defaults to 7 days. All match current docs.)
- M3/1 - Introduction.md

### Unverified or doubtful claims left as they were

- M2/2 line ~61: "From Windows Server 2019, by default, Azure clusters use a DNN instead of a VNN" (for the cluster network name). This is from the Learn unit and matches Windows Server 2019 behaviour as I understand it, but the Azure SQL VM docs I read didn't state it.
- M2/2: "No static IP reservation: Azure DHCP manages IPs dynamically — check for address conflicts manually". The wording is vague; not contradicted by the docs I read.
- M2/3: "Distributed AG: each region needs its own load balancer configured with port 5022". This applies only to VNN/load-balancer designs. Left as is.
- M1/4: ASR "RTO generally within 2 hours". The Site Recovery SLA guarantees a 2-hour RTO for Azure-to-Azure failover. I found this only on the Azure China SLA mirror and in an Azure update post, not on the global SLA page. Left as is.
- M1/6: "Multi-region AG can work in Standard or Enterprise Edition". Standard only supports Basic AGs (2 replicas, 1 DB), so a combined HA+DR AG in Standard is very limited. Left as is because this is the Learn unit's wording.
- M2/6: "System Center Management Pack for Azure SQL" availability not re-checked.
- The SQL DB SLA numbers come from the 2019 Azure blog and SLA mirrors. I couldn't load the live global SLA page.

---

