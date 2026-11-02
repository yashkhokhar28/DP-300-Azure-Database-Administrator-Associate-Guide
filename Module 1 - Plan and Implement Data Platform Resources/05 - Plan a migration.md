# Plan a migration

> **Exam skills:** Evaluate requirements for a migration, Evaluate offline or online migration strategies
> **Reading time:** ~8 min

## The big idea
A good migration is mostly planning. You find what you have, check what will break, pick a target, and decide how much downtime the business can accept. That last choice decides **online** or **offline**, and that decides the tool.

Analogy: moving house. First list your furniture. Check what fits in the new place. Then choose: move everything on one weekend (offline), or move room by room while still living there (online).

## The five stages
| Stage | What you do |
|-------|-------------|
| **1. Discover** | Find every SQL Server, version, database, size and the apps that use them. Get management support. |
| **2. Assess** | Find **blockers** (unsupported features), size and growth. Decide what to merge or retire. |
| **3. Plan** | Pick the target, the migration type, online or offline, the tool and the order. |
| **4. Transform and optimize** | Fix incompatible code, upgrade, right-size. Example: move SSIS packages to Azure Data Factory. |
| **5. Migrate, validate, remediate** | Run the move, test data and apps, fix problems, test DR and monitoring. |

Tips: start with **non-critical** databases. Always do a **test migration** first. Have a **rollback plan**.

**Example:** A retailer bought three companies. It has 40 SQL Servers on five versions. Nobody knows all the apps. Discover and assess come first, before any tool is chosen.

## Migration types (the "R"s)
- **Rehost:** lift and shift to a VM. No changes.
- **Refactor:** small changes to fit PaaS (for example MI).
- **Rearchitect:** change the app to use cloud features.
- **Rebuild / replace / retire:** big change, or remove the app.

## What to evaluate
- **Features used:** SQL Agent, CLR, linked servers, FILESTREAM, cross-database queries, Service Broker. These decide SQL Database vs MI vs VM.
- **Size and growth:** over 4 TB → Hyperscale or MI/VM.
- **Downtime allowed:** minutes, hours, or none.
- **Source version and edition:** some tools need SQL Server 2016+ or Enterprise/Standard.
- **Network:** VPN or ExpressRoute? Can the source reach Azure? Bandwidth for big backups?
- **Server-level objects:** logins, Agent jobs, linked servers. Database-level tools don't move them.
- **Security and compliance:** data that must stay on-premises.
- **HA/DR and performance needs** after the move.

## Assessment tools
- **Azure Migrate:** discover and assess servers and databases, with target sizing.
- **SQL Server migration assessment in Azure Arc:** for Arc-enabled servers. Runs automatically every weekend. Shows blockers and a right-sized MI.
- **SSMS migration component:** for version upgrades.
- Note: **Data Migration Assistant (DMA)** retired in July 2025. It did compatibility checks and small schema/data moves. Old exam questions may still name it.
- Note: **Azure Data Studio** and its **Azure SQL migration extension** retired on **28 Feb 2026**. It drove DMS from a wizard. Its jobs now run from the **DMS portal, PowerShell or CLI**, **Arc migration** or SSMS.
- Note: **Database Experimentation Assistant** retired 15 Dec 2024. **DMS (classic)** SQL scenarios retired 15 Mar 2026. Use the current DMS.

## Online vs offline
| | Offline | Online |
|-|---------|--------|
| Source during copy | Stopped for writes | Keeps working |
| Downtime | The **whole** copy | Only the **cutover** |
| Complexity | Low | Higher |
| Good for | Small databases, weekend windows | 24/7 systems |

**Cutover** = the moment you stop writes on the source, let the last changes arrive, and point apps at the new database.

**Example:** An HR database is quiet at weekends → offline, Friday night to Monday. A hospital's patient system runs 24/7 → online.

## Tool chooser
| Target | Offline tools | Online tools |
|--------|---------------|--------------|
| **SQL Database** | **DMS** (offline only), BACPAC/SqlPackage, bcp, Azure Data Factory | **Transactional replication** |
| **Managed Instance** | Native backup to URL + restore, LRS autocomplete mode, BACPAC via SqlPackage | **MI link**, **LRS continuous**, **Arc migration** (uses link or LRS), DMS, transactional replication |
| **SQL Server on VM** | Backup/restore, detach/attach, BACPAC, bcp, VHD upload, Azure Import/Export | **Distributed AG**, Always On AG, log shipping, transactional replication, DMS online, Azure Migrate |

Key traps:
- **DMS to SQL Database is offline only.** Online to SQL Database = transactional replication.
- **LRS and MI link are MI only.**
- BACPAC is always offline and does not move logins or jobs.
- Partial data (some tables only) → **bcp** or **Azure Data Factory**.
- Full machine as-is (OS + SQL) → **Azure Migrate** server migration.

## Speed tips for any migration
- Fix IO bottlenecks on the source.
- **Scale the target up** during the copy (for example Business Critical or Hyperscale), **scale down** after.
- Make sure network bandwidth is enough.
- Keep files (BACPAC, backups) in the **same region** as the target.
- Use **compressed** backups and **CHECKSUM**.
- Turn off auto-create and auto-update statistics during the load, then turn them back on.
- Apps should use **retry logic** with growing waits (5 s, 10 s, 20 s, up to 60 s) and a new connection after a failure.

## Remember for the exam
- Stages: **Discover → Assess → Plan → Transform/Optimize → Migrate/Validate/Remediate**.
- "Zero or minimal downtime" → online tool. "Weekend window OK" → offline tool.
- "Online to Azure SQL Database" → **transactional replication** (not DMS).
- "Online to MI" → **MI link** or **LRS continuous**.
- "Online to VM" → **distributed AG** or DMS online.
- DMA (July 2025) and Azure Data Studio (28 Feb 2026) are **retired**. Know what they did, pick current tools.
- Database-level tools never move **logins, Agent jobs, linked servers**. Script them.

## Quick check
1. A 24/7 stock system must move to Azure SQL Database with only a few minutes of downtime. Which method?
   <details><summary>Answer</summary>Transactional replication. DMS to SQL Database is offline only.</details>
2. In which stage do you find that a database uses FILESTREAM?
   <details><summary>Answer</summary>Assess. It is a blocker for MI and SQL Database, so the target becomes a VM.</details>
3. An old question says "use DMA to assess". What do you use today?
   <details><summary>Answer</summary>Azure Migrate or the Azure Arc migration assessment. DMA retired in July 2025.</details>
