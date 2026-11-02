# DP-300: Administering Microsoft Azure SQL Solutions — Verified Question Bank

**230 unique questions**, every answer checked against Microsoft Learn. Only questions in the **current DP-300 skills outline (as of 27 Oct 2026)** are included.

| | |
|---|---|
| Sources merged | `DP-300/DP-300.md` (224 Qs, exam-dumps.net), `dp-300.pdf` (CertyIQ, 80 Qs), `DP-300-demo (1).pdf` (CertsHero, 5 Qs), `2025-09/` screenshots (85, 2passeasy, hand-marked). `DP-300.zip` is an identical copy of `DP-300/`. The Zacker Exam Ref PDF was excluded as requested. |
| Raw → unique | 394 transcribed questions → 286 unique → **230 in current scope** (54 excluded, listed at the end) |
| Source answers corrected | **63** questions where at least one source's answer was wrong (see **Source-answer check** on each question) |
| Confidence | High 171 · Medium 48 · Low 11 |

**How to read each question**
- **✅ Answer** — the verified correct answer. **Why** — short explanation.
- **Source-answer check** — what each dump/your notes said: ✔ = correct, ✘ = wrong.
- **Confidence** — *High*: confirmed by Microsoft docs, unambiguous. *Medium*: well supported but the wording or an old exam key leaves room for argument. *Low*: genuinely disputed or the source question was incomplete; both readings are explained.
- Source IDs: `MD-n` = DP-300.md question n · `CQ-n` = CertyIQ PDF · `DEMO-n` = demo PDF · `NOTE-n` = screenshot "NEW QUESTION n" (`imgNNN` = screenshot file number).
- Naming: Azure AD = Microsoft Entra ID. Questions about retired tooling (e.g. DMS classic, DMA) are kept only when the skill is still tested; the explanation notes the current equivalent.

## Contents
- [D1 — Plan and implement data platform resources (15–20%)](#d1--plan-and-implement-data-platform-resources-1520) — 65 questions
- [D2 — Implement a secure environment (20–25%)](#d2--implement-a-secure-environment-2025) — 65 questions
- [D3 — Monitor, configure, and optimize database resources (20–25%)](#d3--monitor-configure-and-optimize-database-resources-2025) — 59 questions
- [D4 — Configure and manage automation of tasks (15–20%)](#d4--configure-and-manage-automation-of-tasks-1520) — 28 questions
- [D5 — Plan and configure a high availability and disaster recovery (HA/DR) environment (20–25%)](#d5--plan-and-configure-a-high-availability-and-disaster-recovery-hadr-environment-2025) — 13 questions
- [Case studies](#case-studies)
- [Excluded questions (not in current outline)](#excluded-questions-not-in-current-outline)


## D1 — Plan and implement data platform resources (15–20%)

*65 questions*

### Q1. Enable page compression on CHAR(400) table
<sub>ID U-001</sub>

**Sources:** MD-021, CQ-038   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database named DB1. You have a table named Table1 that has 20 columns of type CHAR(400). Row compression for Table1 is enabled. During a database audit, you discover that none of the fields contain more than 150 characters. You need to ensure that you can apply page compression to Table1. What should you do?

**Options:**
A. Configure the columns as sparse.
B. Change the column type to NVARCHAR(MAX).
C. Change the column type to VARCHAR(MAX).
D. Change the column type to VARCHAR(200).

**✅ Answer:** D

**Why:** 20 × CHAR(400) = 8,000 bytes of fixed data; with compression overhead the maximum row size exceeds 8,060 bytes, so compression can't be enabled ("max row size plus compression overhead" rule). VARCHAR(200) still holds the 150-character values and cuts the maximum row size to ~4,000 bytes. Sparse columns are incompatible with data compression; (N)VARCHAR(MAX) data can be pushed off-row, where it isn't compressed (and NVARCHAR doubles the storage).

**Source-answer check:** MD & CQ said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/data-compression/data-compression

---

### Q2. factSales 6bn rows: greatest space reduction
<sub>ID U-002</sub>

**Sources:** CQ-003   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database that contains a table named factSales. FactSales contains the columns shown in the following table.

| Name | Data type |
|---|---|
| SalesID | Int |
| Product | Int |
| Total Number | Numeric(8,4) |
| Tax Number | Numeric(8,4) |
| SalesRep | Varchar(30) |

FactSales has 6 billion rows and is loaded nightly by using a batch process. You must provide the greatest reduction in space for the database and maximize performance. Which type of compression provides the greatest space reduction for the database?

**Options:**
A. page compression
B. row compression
C. columnstore compression
D. columnstore archival compression

**✅ Answer:** D

**Why:** The question asks which type gives the *greatest space reduction*. Columnstore archival compression (COLUMNSTORE_ARCHIVE) applies XPRESS compression on top of columnstore compression, so it gives the smallest footprint. Page and row (rowstore) compression give far lower ratios than columnstore on a 6-billion-row fact table. Plain columnstore compression is faster to query but compresses less than archival.

**Source-answer check:** CQ said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/data-compression/data-compression

---

### Q3. Partitioned columnstore: max ratio, indexable compression
<sub>ID U-003</sub>

**Sources:** MD-023, CQ-040   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription that contains an Azure SQL database. The database contains a table named table1 that uses partitioned columnstores. You need to configure table1 to meet the following requirements:
- Each partition must be compressed.
- The compression ratio must be maximized.
- You must be able to index the compressed data.

What should you use?

**Options:**
A. page compression
B. columnstore compression
C. GZIP compression
D. columnstore archival compression

**✅ Answer:** D

**Why:** All columnstore tables already use columnstore compression. To *maximize* the ratio you add archival compression (COLUMNSTORE_ARCHIVE), which can be set per partition and is still a columnstore index, so the data stays indexed. The "index" requirement is there to rule out GZIP: data compressed with the COMPRESS function can't be indexed. Page compression applies only to rowstore. B is the default and gives a lower ratio than archival.

**Source-answer check:** MD said D ✔; CQ said B ✘ (archival is slower to query, but it is still an indexed columnstore and gives the higher ratio)
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/data-compression/data-compression

---

### Q4. Retained analytical table: maximize storage reduction
<sub>ID U-004</sub>

**Sources:** MD-144   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database. The database contains a table of analytical data that must be retained. You need to implement a data compression solution that maximizes a reduction in storage. Which type of compression should you use?

**Options:**
A. page
B. columnstore archive
C. row
D. columnstore

**✅ Answer:** B

**Why:** COLUMNSTORE_ARCHIVE gives the highest compression ratio and suits data that must be kept but is rarely queried. It costs extra CPU and time on access. Row and page compression (rowstore) and standard columnstore all compress less.

**Source-answer check:** MD said B ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/data-compression/data-compression

---

### Q5. ROW compression effects on table1 columns
<sub>ID U-005</sub>

**Sources:** MD-215   **Type:** Statements(Yes/No)   **Case study:** —

**Question:**
You have an Azure subscription that contains an Azure SQL database named DB1. You execute the following T-SQL statements against DB1.
```sql
CREATE TABLE table1 (
  column1 TINYINT,
  column2 VARCHAR(255),
  column3 NCHAR(10),
  column4 BINARY(50),
  column5 DATETIME2
);

ALTER TABLE table1 REBUILD WITH (DATA_COMPRESSION = ROW);
```
For each of the following statements, select Yes if the statement is true. Otherwise, select No.

**Options:**
S1. All the fields in column3 will occupy 10 bytes on disk.
S2. Dictionary compression will be used to reduce the size of table1 on disk.
S3. The data types stored in column1, column2, and column4 support compression.

**✅ Answer:** S1: No; S2: No; S3: No

**Why:** S1: uncompressed, NCHAR(10) takes 20 bytes (2 bytes/char). Row compression removes trailing padding and applies Unicode compression, so the size varies from row to row; it is not a fixed 10 bytes. S2: dictionary (and prefix) compression is part of PAGE compression, not ROW. S3: in the row-compression table, tinyint is "No" (1 byte is already the minimum) and varchar is "No effect". Only binary (trailing zeros removed) is affected, so the statement is false for the set.

**Source-answer check:** MD answer image said S1 Yes ✘, S2 No ✔, S3 Yes ✘. MD's own explanation text already disputes S1 (20 bytes). S3 is wrong per the Microsoft row-compression data-type table.
**Confidence:** Medium (S2 certain. S1/S3 are doc-based; no community thread was found to confirm what the exam key intended.)   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/data-compression/row-compression-implementation

---

### Q6. SIM: page-compress PK_SalesOrderHeader clustered index
<sub>ID U-006</sub>

**Sources:** MD-210   **Type:** Simulation   **Case study:** —

**Question:**
SIMULATION. In an Azure SQL database named db1, you need to enable page compression on the PK_SalesOrderHeader_SalesOrderID clustered index of the SalesLT.SalesOrderHeader table. To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:**
none (lab task)

**✅ Answer:**
Option 1, T-SQL (SSMS connected to db1):
```sql
ALTER INDEX PK_SalesOrderHeader_SalesOrderID
ON SalesLT.SalesOrderHeader
REBUILD WITH (DATA_COMPRESSION = PAGE);
```
Option 2, SSMS GUI:
1. Connect SSMS to the logical server and expand db1 > Tables > SalesLT.SalesOrderHeader > Indexes.
2. Right-click **PK_SalesOrderHeader_SalesOrderID** > **Storage** > **Manage Compression**.
3. In the Data Compression Wizard, set **Compression type** = **Page** (optionally select **Calculate** to estimate savings) > **Next**.
4. Choose **Run immediately** > **Next** > **Finish**.
5. Verify: `SELECT data_compression_desc FROM sys.partitions WHERE object_id = OBJECT_ID('SalesLT.SalesOrderHeader') AND index_id = 1;` returns PAGE.

**Why:** Page compression on an existing index is applied by rebuilding the index with DATA_COMPRESSION = PAGE (ALTER INDEX … REBUILD). The SSMS Data Compression Wizard generates the same statement. Because this is the clustered index, the table's data is compressed. Nonclustered indexes don't inherit the setting.

**Source-answer check:** MD (wizard, Page, run immediately) ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/data-compression/data-compression

---

### Q7. ADatum: SVR1 to Instance1 migration service/endpoint
<sub>ID U-007</sub>

**Sources:** MD-002   **Type:** Hotspot   **Case study:** ADatum v1 (financial services)

**Question:**
You need to recommend which service and target endpoint to use when migrating the databases from SVR1 to Instance1. The solution must meet the availability requirements. What should you recommend?
(Case facts: SVR1/SVR2 run SQL Server 2016 Enterprise with AG1 containing DB1 and DB2. The datacenter is connected to Network1 by S2S VPN. Instance1 is deployed to Network1. Requirement: minimize downtime during the migration of DB1 and DB2.)

**Options:**
Box 1 – Migration service: Log Replay Service (LRS) | Managed Instance link | SQL Data Sync
Box 2 – Target endpoint: A private endpoint | A public endpoint | A VNet-local endpoint

**✅ Answer:** Box 1: Managed Instance link; Box 2: A VNet-local endpoint

**Why:** The MI link (distributed AG) gives near-real-time replication and the smallest downtime of any MI migration method. It supports SQL Server 2016 SP3 + Azure Connect pack and later. LRS works too but has more cutover downtime, and SQL Data Sync doesn't support MI. Microsoft Learn states the link can be established only with the VNet-local endpoint; public and private endpoints can't be used. The S2S VPN to Network1 provides the private connectivity.

**Source-answer check:** MD said MI link + VNet-local endpoint ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/managed-instance-link-feature-overview

---

### Q8. Litware: SERVER1 migration DMS tier + resource
<sub>ID U-008</sub>

**Sources:** MD-001, DEMO-1   **Type:** Hotspot   **Case study:** Litware

**Question:**
You are planning the migration of the SERVER1 databases. The solution must meet the business requirements. What should you include in the migration plan? To answer, select the appropriate options in the answer area.
(Case facts: SERVER1 is on-premises SQL Server 2012 with two 1-TB databases. Target: Azure SQL Database. Business requirement: minimize downtime during the migration of the SERVER1 databases. The primary datacenter has ExpressRoute with private peering to HubVNet.)

**Options:**
Box 1 – Azure Database Migration Service pricing tier: Standard 2-vCore | Standard 4-vCore | Premium 4-vCore
Box 2 – Required Azure resource: A virtual network that has service endpoints | A VPN gateway | An Azure Logic app

**✅ Answer:** Box 1: **Premium 4-vCore**; Box 2: **A virtual network that has service endpoints**

**Why:** Box 1: In DMS (classic) the Standard tiers do offline migrations only. Premium is required for an *online* (minimal-downtime) migration, which is the business requirement. Box 2: DMS needs an Azure VNet with connectivity to the on-premises source through ExpressRoute or VPN. Litware **already has ExpressRoute** (Microsoft + private peering), so a new VPN gateway adds nothing, and the requirement to minimize on-premises infrastructure argues against one. Microsoft's DMS prerequisites state that when ExpressRoute is used, the DMS subnet needs **service endpoints** (target SQL, Storage, Service Bus), because DMS has no internet connectivity. A Logic app is irrelevant.
Exam-era note: DMS (classic) retired on 15 Mar 2026. This item tests the classic behavior.

**Source-answer check:** MD-001 said Premium 4-vCore ✔ + VPN gateway ✘ (that's the long-circulating dump key; a VPN is only needed when there's no ExpressRoute). DEMO-1's answer image said Standard 4-vCore ✘, which contradicts its own explanation text (Premium).
**Confidence:** Medium (Box 1 High; Box 2 rests on the case's existing ExpressRoute plus the DMS ExpressRoute/service-endpoint prerequisite. Many dumps still show "VPN gateway".)   **Ref:** https://learn.microsoft.com/en-us/azure/dms/pre-reqs ; https://learn.microsoft.com/en-us/azure/dms/tutorial-sql-server-azure-sql-database-offline

---

### Q9. 2-TB SQL 2019 to MI: min downtime/effort
<sub>ID U-013</sub>

**Sources:** MD-038, CQ-055   **Type:** MCQ   **Case study:** —

**Question:**
You have an on-premises datacenter that contains a 2-TB Microsoft SQL Server 2019 database named DB1. You need to recommend a solution to migrate DB1 to an Azure SQL managed instance. The solution must minimize downtime and administrative effort. What should you include in the recommendation?

**Options:**
A. Log Replay Service (LRS)
B. log shipping
C. transactional replication
D. SQL Data Sync

**✅ Answer:** A

**Why:** LRS is the managed, log-shipping-based MI migration service. It restores full/diff/log backups from Blob Storage continuously and then cuts over, so downtime is low with little setup. You can't configure classic log shipping to MI yourself (no manual NORECOVERY restores). Transactional replication is an "alternative" option with relatively complex setup and per-table limitations. SQL Data Sync doesn't support MI.

**Source-answer check:** MD & CQ said A ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/data-migration/sql-server/managed-instance/overview

---

### Q10. 4-TB SQL 2019 to MI: min downtime/data loss
<sub>ID U-014</sub>

**Sources:** CQ-005   **Type:** MCQ   **Case study:** —

**Question:**
You have a Microsoft SQL Server 2019 instance in an on-premises datacenter. The instance contains a 4-TB database named DB1. You plan to migrate DB1 to an Azure SQL Database managed instance. What should you use to minimize downtime and data loss during the migration?

**Options:**
A. distributed availability groups
B. database mirroring
C. Always On Availability Group
D. Azure Database Migration Service

**✅ Answer:** D

**Why:** DMS performs *online* migrations to SQL MI: it continuously restores log backups, and downtime and data loss are limited to the final cutover. You can't configure database mirroring or a regular AG with MI as a replica. A distributed AG to MI exists only inside the Managed Instance link feature, not as a stand-alone feature you configure. With DMS in the list, D is the intended answer. Note: a different variant of this question has no DMS option (choices: distributed AGs / mirroring / log shipping / DMA). In that variant, "distributed availability groups" (= MI link technology) would be the answer.

**Second opinion:** Disputed: the MI link (built on **distributed availability groups**, option A) gives the lowest downtime and can fail over with no data loss. Some keys and independent reviewers pick A, while the CertyIQ and SPOTO keys pick D (DMS online migration). Both meet the goal. D is the published key here.

**Source-answer check:** CQ said D ✔
**Confidence:** Low (some argue A because the MI link uses distributed AGs)   **Ref:** https://learn.microsoft.com/en-us/azure/dms/dms-overview ; https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/managed-instance-link-feature-overview

---

### Q11. 60-TB over 10 Mbps: fastest migration
<sub>ID U-015</sub>

**Sources:** MD-060, MD-115   **Type:** MCQ   **Case study:** —

**Question:**
Your on-premises network contains a server that hosts a 60-TB database named DB1. The network has a 10-Mbps internet connection. You need to migrate DB1 to Azure. The solution must minimize how long it takes to migrate the database. What should you use?

**Options:**
A. Data Migration Assistant (DMA)
B. Azure Migrate
C. Azure Data Studio
D. Azure Data Box

**✅ Answer:** D

**Why:** Sending 60 TB over a 10-Mbps link takes well over a year (~1.5 years at full line rate), so every online tool (DMA, Azure Migrate, Azure Data Studio/DMS) is limited by the link. Azure Data Box is Microsoft's offline, shipped appliance for transfers larger than 40 TB with no or limited network connectivity. You ship the backups/data on the device, then restore in Azure.

**Variant seen in other dumps (MD-115):** same question with options `A. Azure Migrate / B. Azure Data Box / C. Azure Database Migration Service / D. Data Migration Assistant (DMA)` → answer **B**.

**Source-answer check:** MD said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/databox/data-box-overview

---

### Q12. Backup to block blob for MI: max security
<sub>ID U-017</sub>

**Sources:** MD-105   **Type:** MCQ   **Case study:** —

**Question:**
You have an on-premises Microsoft SQL Server 2019 instance named SQL1 that hosts a database named db1. You have an Azure subscription that contains an Azure SQL managed instance named MI1 and an Azure Storage account named storage1. You plan to migrate db1 to MI1 by using the backup and restore process. You need to ensure that you can back up db1 to storage1. The solution must meet the following requirements:
- Use block blob storage.
- Maximize security.

What should you do on storage1?

**Options:**
A. Generate a shared access signature (SAS).
B. Create an access policy.
C. Rotate the storage keys.
D. Enable infrastructure encryption.

**✅ Answer:** A

**Why:** BACKUP TO URL writes block blobs only when the SQL credential holds a SAS token. A storage account key in the credential produces page blobs. Microsoft calls SAS the safer way to authorize blob access compared with the storage key, because it is scoped and time-limited. An access policy alone doesn't give a credential. Key rotation and infrastructure encryption don't enable backup to URL.

**Source-answer check:** MD said A ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/sql-server-backup-to-url

---

### Q13. Copy MI database DB2 to SQL Server 2022
<sub>ID U-018</sub>

**Sources:** MD-067   **Type:** DragDrop   **Case study:** —

**Question:**
You have an on-premises server named Server1 that has Microsoft SQL Server 2022 installed. You have an Azure subscription that contains an Azure SQL managed instance named MI1. MI1 contains a database named DB2. You need to create a copy of DB2 on Server1. Which three actions should you perform in sequence?

**Options:**
Actions:
- Back up DB2 to a file and use the COPY_ONLY option.
- Back up DB2 to a URL and use the COPY_ONLY option.
- Restore DB2 from the file and use the RESTORE WITH RECOVERY option.
- Create a credential on MI1.
- Create a credential on Server1.
- Restore DB2 from the URL and use the WITH MOVE option.

**✅ Answer:** 1. Create a credential on MI1 → 2. Back up DB2 to a URL and use the COPY_ONLY option → 3. Restore DB2 from the URL and use the WITH MOVE option

**Why:** This follows the Microsoft Learn procedure "Restore a database to SQL Server from SQL MI": create a credential on the MI (managed identity or SAS) for the storage container, take a COPY_ONLY backup TO URL, then RESTORE … FROM URL WITH MOVE on SQL Server 2022 (WITH MOVE with explicit file paths is mandatory). MI can't back up to a local file, so the "file" actions are wrong. Server1 also needs read access to the blob (a credential there) in practice, but the three-step answer the question expects is the documented sequence above. Prerequisite: MI1 must use the SQL Server 2022 update policy.

**Source-answer check:** MD said credential on MI1 → backup to URL COPY_ONLY → restore from URL WITH MOVE ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/restore-database-to-sql-server

---

### Q14. DMS over ExpressRoute to SQLMI1: VNET1 config
<sub>ID U-019</sub>

**Sources:** MD-018, CQ-035, NOTE-142   **Type:** MCQ   **Case study:** —

**Question:**
You have an on-premises Microsoft SQL Server 2019 server that hosts a database named DB1. You have an Azure subscription that contains an Azure SQL managed instance named SQLMI1 and a virtual network named VNET1. SQLMI1 resides on VNET1. The on-premises network connects to VNET1 by using an ExpressRoute connection. You plan to migrate DB1 to SQLMI1 by using Azure Database Migration Service. You need to configure VNET1 to support the migration. What should you do?

**Options:**
A. Configure service endpoints.
B. Configure virtual network peering.
C. Deploy an Azure firewall.
D. Configure network security groups (NSGs).

**✅ Answer:** A

**Why:** The DMS (classic) prerequisites said: if you use ExpressRoute with network peering to Microsoft, add service endpoints (target database/Microsoft.Sql, Storage, Service Bus) to the subnet where DMS is provisioned, because DMS has no direct internet connectivity. VNet peering isn't needed because SQLMI1 and DMS are on the same VNET1 and on-prem connectivity already exists through ExpressRoute. NSGs only need to *not block* the required ports (443, 1433, etc.); creating NSGs isn't the required configuration. An Azure Firewall isn't required.

**Source-answer check:** CQ said A ✔; MD said D ✘ (NSGs only need to allow traffic; they don't enable the scenario); NOTE said B ✘ (no second VNet to peer)
**Confidence:** Medium (classic-DMS-era prerequisite; DMS classic was retired 15 Mar 2026)   **Ref:** https://learn.microsoft.com/en-ca/azure/dms/resource-network-topologies

---

### Q15. Five SQL1 databases to MI: no data loss
<sub>ID U-020</sub>

**Sources:** MD-022, CQ-039   **Type:** MCQ   **Case study:** —

**Question:**
You have an on-premises Microsoft SQL Server named SQL1 that hosts five databases. You need to migrate the databases to an Azure SQL managed instance. The solution must minimize downtime and prevent data loss. What should you use?

**Options:**
A. Always On availability groups
B. Backup and Restore
C. log shipping
D. Database Migration Assistant

**✅ Answer:** B

**Why:** Of the listed options, only native backup and restore (full .bak from Azure Blob Storage, RESTORE FROM URL) is a supported migration method for SQL MI. Microsoft calls it "the easiest migration option", and with the source quiesced before the final backup there's no data loss. You can't add MI as a replica in a regular Always On AG; AG technology reaches MI only through the Managed Instance link feature. Manual log shipping isn't possible because MI doesn't allow user RESTORE … WITH NORECOVERY; the managed equivalent is LRS. DMA only assesses and can't migrate to MI. If "Managed Instance link", "LRS" or "DMS" were offered, they would give lower downtime.

**Second opinion:** Disputed: with the Managed Instance link (Always On AG technology, option A), downtime is near zero. Pass4Success marks C, while MD, CertyIQ and SPOTO mark B. B is the most widely published key and the only classic, fully supported method in the list.

**Source-answer check:** MD & CQ said B ✔
**Confidence:** Low (weak option set; some argue A or C as stand-ins for the MI link and LRS, but neither is literally supported for MI)   **Ref:** https://learn.microsoft.com/en-us/data-migration/sql-server/managed-instance/overview

---

### Q16. Hyperscale to Business Critical: first step
<sub>ID U-021</sub>

**Sources:** MD-073   **Type:** Hotspot   **Case study:** —

**Question:**
You have two Azure SQL databases named SQLDB1 and SQLDB2 in the Hyperscale service tier. SQLDB1 was set to the Hyperscale service tier when the database was created. SQLDB2 was changed from the Business Critical service tier to the Hyperscale service tier during the last 14 days. You need to change SQLDB1 and SQLDB2 to the Business Critical service tier. The solution must minimize the administrative effort. What should you do first?

**Options:**
On SQLDB1: Export the data. | Set Service tier to Business Critical. | Set Service tier to General Purpose.
On SQLDB2: Export the data. | Set Service tier to Business Critical. | Set Service tier to General Purpose.

**✅ Answer:** SQLDB1: Export the data; SQLDB2: Set Service tier to General Purpose

**Why:** Databases originally created in Hyperscale aren't eligible for reverse migration. The only way out is export/import (bacpac or other data movement), so SQLDB1 must be exported first. SQLDB2 was converted within the 45-day window, so it can be reverse migrated. Reverse migration targets General Purpose only; you then change GP → Business Critical as a normal scaling operation. A direct change to Business Critical isn't allowed for either database.

**Source-answer check:** MD said SQLDB1 Export / SQLDB2 General Purpose ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/reverse-migrate-from-hyperscale

---

### Q17. LRS cmdlet with most secure storage auth
<sub>ID U-022</sub>

**Sources:** MD-213   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure SQL managed instance named Server1 and an Azure Blob Storage account named storage1 that contains Microsoft SQL Server database backup files. You plan to use Log Replay Service to migrate the backup files from storage1 to Server1. The solution must use the highest level of security when connecting to storage1. Which PowerShell cmdlet should you run, and which parameter should you specify to secure the connection?

**Options:**
Cmdlet: Complete-AzSqlInstanceDatabaseLogReplay | Get-AzSqlInstanceDatabaseLogReplay | Start-AzSqlInstanceDatabaseLogReplay
Parameter: ManagedIdentity | SharedAccessSignature | StorageAccountKey

**✅ Answer:** Cmdlet: Start-AzSqlInstanceDatabaseLogReplay; Parameter: ManagedIdentity

**Why:** Start-AzSqlInstanceDatabaseLogReplay starts the LRS restore. Its -StorageContainerIdentity parameter takes ManagedIdentity or SharedAccessSignature. ManagedIdentity (the MI's identity granted Storage Blob Data Reader on the container) needs no secret or token, so it's the most secure. SharedAccessSignature needs a SAS token (a secret). StorageAccountKey isn't an LRS auth option. Get- only monitors and Complete- only does the cutover.

**Source-answer check:** MD said Start-… + ManagedIdentity ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/powershell/module/az.sql/start-azsqlinstancedatabaselogreplay ; https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/log-replay-service-migrate

---

### Q18. Monitor MI-to-MI database copy performance
<sub>ID U-023</sub>

**Sources:** MD-216   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription that contains two Azure SQL managed instances named SQLMI1 and SQLMI2. SQLMI1 hosts a database named DB1. You initiate a copy operation of DB1 to SQLMI2. You need to monitor the performance of the copy operation. Which dynamic management view should you use?

**Options:**
A. sys.dm_continuous_copy_status
B. sys.dm_database_replica_states
C. sys.dm_hadr_physical_seeding_stats
D. sys.dm_db_mirroring_connections

**✅ Answer:** C

**Why:** MI database copy/move uses Always On seeding. Microsoft Learn says the "seeding process and its speed can be monitored via DMV sys.dm_hadr_physical_seeding_stats" (transfer_rate_bytes_per_second, estimate_time_complete_utc, …). sys.dm_continuous_copy_status is for Azure SQL Database geo-replication / database copy. sys.dm_database_replica_states shows AG replica sync state, not seeding throughput. sys.dm_db_mirroring_connections is for database mirroring connections.

**Source-answer check:** MD said C ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/database-copy-move-how-to

---

### Q19. Offline MI migration via ADS: install on SQL1
<sub>ID U-024</sub>

**Sources:** MD-063   **Type:** MCQ   **Case study:** —

**Question:**
You have an on-premises Microsoft SQL Server instance named SQL1 that hosts multiple databases. You have an Azure subscription that contains an Azure SQL managed instance named SQLMI1. You need to perform an offline migration of the SQL1 databases to SQLMI1 by using Azure Data Studio. What should you install on SQL1?

**Options:**
A. Data Migration Assistant (DMA)
B. SQL Server Data Tools (SSDT)
C. a self-hosted integration runtime
D. an Azure Migrate appliance

**✅ Answer:** C

**Why:** The Azure SQL Migration extension (Azure Data Studio) uses Azure DMS. When backups are on an on-premises SMB share, DMS needs a self-hosted integration runtime (SHIR) that can reach the source SQL Server and the share and uploads the backups to Azure Storage. DMA is an assessment tool. SSDT is a dev tool. The Azure Migrate appliance does discovery/assessment. Note: Microsoft recommends a separate machine near the source for the SHIR, but of these options it's the only component required. Azure Data Studio was retired on 28 Feb 2026; the same DMS + SHIR requirement now applies through the Azure portal/PowerShell/CLI.

**Source-answer check:** MD said C ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/data-migration/sql-server/self-hosted-integration-runtime ; https://learn.microsoft.com/en-us/AZURE/dms/migration-using-azure-data-studio

---

### Q20. Minimal-downtime migration to Azure SQL Database
<sub>ID U-025</sub>

**Sources:** MD-020, CQ-037   **Type:** MCQ   **Case study:** —

**Question:**
You need to migrate an on-premises Microsoft SQL Server database to Azure SQL Database. The solution must minimize downtime. What should you do?

**Options:**
A. Configure Transaction Log Shipping.
B. Implement Always On availability groups.
C. Configure transactional replication.
D. Import a BACPAC.

**✅ Answer:** C

**Why:** Azure SQL Database can be a push subscriber of one-way transactional replication from SQL Server; Microsoft documents this as the data-migration scenario (replicate, redirect apps, drop the publication), so downtime is limited to the cutover. Log shipping and AGs can't target Azure SQL Database (they restore/seed a full instance database), and a BACPAC import is an offline method with downtime for the whole export/import.

**Source-answer check:** MD & CQ said C ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/replication-to-sql-database

---

### Q21. DMS online migration: backup type and option
<sub>ID U-026</sub>

**Sources:** CQ-006   **Type:** Hotspot   **Case study:** —

**Question:**
You have an on-premises Microsoft SQL Server 2016 server named Server1 that contains a database named DB1. You need to perform an online migration of DB1 to an Azure SQL Managed Instance by using Azure Database Migration Service. How should you configure the backup of DB1?

**Options:**
Backup type: Full and log backups only | Full backup only | Log backup only
Backup option: WITH CHECKSUM | WITH NOINIT | WITH UNLOAD

**✅ Answer:** Backup type: Full and log backups only; Backup option: WITH CHECKSUM

**Why:** An online (continuous restore) migration needs a full backup plus subsequent log backups that DMS keeps restoring until cutover. Backups should be taken WITH CHECKSUM (classic DMS required it; current DMS validates checksums automatically). WITH NOINIT appends to existing media, which isn't supported (each backup must be in its own file), and WITH UNLOAD is a tape option.

**Source-answer check:** CQ said Full and log / WITH CHECKSUM ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/dms/faq ; https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/log-replay-service-migrate

---

### Q22. DMS online migration: minimize backup files
<sub>ID U-027</sub>

**Sources:** MD-030, CQ-047   **Type:** Hotspot   **Case study:** —

**Question:**
You have an on-premises Microsoft SQL Server 2016 instance that hosts a database named db1. You have an Azure subscription that contains an Azure SQL managed instance named MI1. You plan to perform an online migration of db1 to MI1 by using Azure Database Migration Service. You need to create the backups for the migration. The solution must minimize the number of backup files created. Which type of backups should you create, and how should you store the backups?

**Options:**
Backup type: Full only | Full and differential | Full and transaction log | Transaction log only
To store the backups: Append all the backups to a single file. | Append each backup to a separate file. | Write each backup to a separate file.

**✅ Answer:** Backup type: Full and transaction log; To store the backups: Write each backup to a separate file.

**Why:** Online migration requires a full backup followed by log backups (differentials add files without being required). DMS doesn't support appending multiple backups (e.g., full and log) into a single backup media, so the minimum is one file per backup ("write each backup to a separate file"); "append each backup to a separate file" is a distractor wording that still implies append semantics.

**Source-answer check:** MD & CQ agree ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/dms/faq

---

### Q23. DMS online migration: backup command and location
<sub>ID U-028</sub>

**Sources:** MD-016, CQ-033   **Type:** Hotspot   **Case study:** —

**Question:**
You have an on-premises Microsoft SQL Server 2019 instance that hosts a database named DB1. You plan to perform an online migration of DB1 to an Azure SQL managed instance by using Azure Database Migration Service. You need to create a backup of DB1 that is accessible to Azure Database Migration Service. What should you run for the backup and where should you store the backup?

**Options:**
Run: A full backup and a log backup appended to the same file by using the WITH CHECKSUM option | A full backup and a log backup to separate files by using the WITH CHECKSUM option | A full backup and a log backup to separate files by using the WITH FILE_SNAPSHOT option
Store the backup in: A Recovery Services vault | An Azure Blob storage account | An SMB file share

**✅ Answer:** Run: A full backup and a log backup to separate files by using the WITH CHECKSUM option; Store the backup in: An SMB file share

**Why:** DMS restores native backups that must be in separate files (appending to one media isn't supported) and taken WITH CHECKSUM; FILE_SNAPSHOT is only for databases whose files are already in Azure Blob storage. Classic DMS (the exam-era service) required the backups on an SMB network share it could reach. Note: current DMS (Azure SQL migration extension/portal) accepts either an SMB share or an Azure Blob container, so "Azure Blob storage account" would also work today, but a Recovery Services vault never does. Keep SMB file share as the exam answer.

**Source-answer check:** MD & CQ agree (SMB share) ✔ — classic-key answer; Blob is also valid under current DMS.
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/azure/dms/faq

---

### Q24. Replication type to Azure SQL Database
<sub>ID U-029</sub>

**Sources:** MD-040, CQ-057   **Type:** MCQ   **Case study:** —

**Question:**
You have an on-premises Microsoft SQL Server 2019 instance that hosts a database named DB1. You have an Azure subscription that contains an Azure SQL database named SQLDB1. You need to replicate DB1 to SQLDB1. Which type of replication should you use?

**Options:**
A. transactional
B. peer-to-peer
C. snapshot
D. merge

**✅ Answer:** A

**Why:** Azure SQL Database supports only standard (one-way) transactional and snapshot replication, and only as a push subscriber. Merge and peer-to-peer aren't supported. Snapshot only copies point-in-time data and doesn't keep the subscriber in sync, so continuous replication of DB1 means transactional.

**Source-answer check:** MD & CQ said A ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/replication-to-sql-database

---

### Q25. Replication to SQLDB1: subscriber role and type
<sub>ID U-030</sub>

**Sources:** MD-065   **Type:** Hotspot   **Case study:** —

**Question:**
You have an on-premises Microsoft SQL Server instance named SQLSVR1 that hosts a database named DB1. You have an Azure subscription that contains an Azure SQL database named SQLDB1. You need to migrate the data stored in DB1 to SQLDB1 by using SQL Server replication. The solution must minimize the performance impact on DB1. How should you configure the replication?

**Options:**
Configure SQLDB1: As a push subscriber only | As a push subscriber and a distributor | To use a pull subscription only
Type of replication: Snapshot | Transactional with peer-to-peer replication | Transactional with updatable subscriptions

**✅ Answer:** Configure SQLDB1: As a push subscriber only; Type of replication: Snapshot

**Why:** Azure SQL Database can only be a push subscriber. The distribution database and replication agents can't be placed in Azure SQL Database, and pull subscriptions aren't supported. Of the listed types, only snapshot is supported: Azure SQL Database doesn't support peer-to-peer or updatable subscriptions (immediate/queued updating). Plain one-way transactional replication would be ideal, but it isn't among the choices.

**Source-answer check:** MD said "push subscriber and a distributor" + "Transactional with updatable subscriptions" ✘. Both are explicitly unsupported for Azure SQL Database (MD's own explanation text even says on-premises is the distributor). No community key found online.
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/replication-to-sql-database

---

### Q26. AG on Windows SQL 2017 to Linux SQL 2019
<sub>ID U-031</sub>

**Sources:** MD-041, CQ-058   **Type:** Hotspot   **Case study:** —

**Question:**
You have two on-premises servers that run Windows Server 2019 and host a Microsoft SQL Server 2017 Always On availability group named AG1. AG1 contains a single database named DB1. You have an Azure subscription. The subscription contains a virtual machine named VM1 that runs Linux. You need to migrate DB1 to a SQL Server 2019 instance on VM1. The solution must minimize the downtime of DB1 during the migration. What should you do?

**Options:**
To prepare for the migration: Add a secondary replica to AG1. | Create a SQL Server 2019 Always On availability group on VM1. | Upgrade the on-premises SQL servers to SQL Server 2019.
To perform the migration, use: A distributed availability group | Azure Migrate | Log shipping

**✅ Answer:** Prepare: Create a SQL Server 2019 Always On availability group on VM1; Perform: A distributed availability group

**Why:** A distributed AG spans two separate AGs and works cross-platform (Windows WSFC AG and Linux AG). From SQL Server 2017 on it can also mix major versions when the primary AG runs the lower version. Microsoft documents this as the migration/upgrade scenario, with downtime limited to the final failover (seeding must be manual for a higher-version target). It needs an AG on each side, so you create an AG on VM1. Adding a SQL 2019 Linux replica to the Windows 2017 AG1 isn't a supported long-term mixed-version, cross-cluster setup. Azure Migrate does lift-and-shift of whole servers and can't move a database into an existing Linux SQL 2019 instance. Log shipping works but needs a longer cutover.

**Source-answer check:** MD & CQ said "Add a secondary replica to AG1" + "Azure Migrate" ✘. Azure Migrate can't move DB1 into the existing SQL 2019 instance on Linux VM1; the cross-platform, cross-version, minimal-downtime method is a distributed AG.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/distributed-availability-groups?view=sql-server-ver17 ; https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-availability-group-cross-platform?view=sql-server-ver16

---

### Q27. Feature blocking migration to Azure SQL Database
<sub>ID U-032</sub>

**Sources:** CQ-004, NOTE-007   **Type:** MCQ   **Case study:** —

**Question:**
You have a Microsoft SQL Server 2019 database named DB1 that uses the following database-level and instance-level features:
- Clustered columnstore indexes
- Automatic tuning
- Change tracking
- PolyBase

You plan to migrate DB1 to an Azure SQL database. What feature should be removed or replaced before DB1 can be migrated?

**Options:**
A. Clustered columnstore indexes
B. PolyBase
C. Change tracking
D. Automatic tuning

**✅ Answer:** B

**Why:** Azure SQL Database supports columnstore indexes (S3 and above, and all vCore tiers), change tracking and automatic tuning. PolyBase is an instance-level SQL Server feature (external data sources such as Hadoop, Oracle, Teradata and other SQL Servers) that Azure SQL Database doesn't provide. Its only replacement there is limited "data virtualization" (OPENROWSET and external tables over CSV, Parquet or Delta files in Azure Storage), so PolyBase usage has to be replaced.

**Source-answer check:** CQ & NOTE said B ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/features-comparison

---

### Q28. Tools to assess SQL Server for Azure targets
<sub>ID U-033</sub>

**Sources:** MD-066   **Type:** MultiSelect   **Case study:** —

**Question:**
You have eight on-premises servers that have Microsoft SQL Server 2022 installed. Each server contains multiple databases. You plan to migrate the on-premises databases to Azure. You need to analyze the servers to identify which Azure SQL services can be used to host the databases. Which two tools can you use? Each correct answer presents a complete solution.

**Options:** (The option list is missing in the source and wasn't found online, so it is RECONSTRUCTED. The source explanation fixes A = Data Migration Assistant and D = Azure Migrate.)
A. Data Migration Assistant (DMA)
B. SQL Server Migration Assistant (SSMA) *(reconstructed)*
C. Azure Database Migration Service *(reconstructed)*
D. Azure Migrate
E. *(unknown)*

**✅ Answer:** A, D (exam key). With current tooling: Azure Migrate, plus SQL Server enabled by Azure Arc (migration assessment) in place of DMA.

**Why:** Azure Migrate discovers SQL Server instances at scale and produces readiness and right-sized recommendations for Azure SQL Database, SQL Managed Instance and SQL Server on Azure VMs. DMA assessed compatibility and recommended Azure SQL targets, so it was the classic second answer. However, Microsoft retired DMA on 16 July 2025 and points to Azure Arc-enabled SQL Server migration assessments and Azure Migrate instead. SSMA is for non-SQL Server sources, and DMS performs migrations rather than assessing target options.

**Source-answer check:** MD said A, D, which is correct for the exam key ✔. Note that DMA is now retired: if Azure Arc appears as an option on a refreshed exam, prefer Azure Migrate + Azure Arc.
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/sql/sql-server/azure-arc/migration-overview?view=sql-server-ver17 ; https://techcommunity.microsoft.com/blog/microsoftdatamigration/announcing-retirement-of-microsoft-data-migration-assistant-dma-tool/4424400

---

### Q29. 14-TB database: SQL MI tier/hardware support
<sub>ID U-035</sub>

**Sources:** CQ-062, MD-045   **Type:** MultiSelect   **Case study:** —

**Question:**
You have an on-premises datacenter that contains a 14-TB Microsoft SQL Server database. You plan to create an Azure SQL managed instance and migrate the on-premises database to the new instance. Which three service tiers support the SQL managed instance? Each correct answer presents a complete solution.

**Options:** (from CQ-062; MD-045 had no option list)
A. General Purpose Standard
B. Business Critical Memory Optimized Premium
C. General Purpose Premium
D. Business Critical Premium
E. Business Critical Standard

**✅ Answer:** A, B, C

**Why:** Max instance storage per the current resource limits:
- General Purpose: 16 TB on all hardware (32 TB on Next-gen GP).
- Business Critical, standard-series: 4 TB, so it can't hold the database.
- Business Critical, memory optimized premium-series: 16 TB at 64 or more vCores.
- Business Critical, premium-series: 5.5 TB in most regions, but 16 TB at 64 or more vCores in the major regions.

The intended three are GP standard-series, GP premium-series and BC memory-optimized premium-series. Caveat: in major regions, BC premium-series (D) can now also reach 16 TB, so D is technically possible today. The regionally guaranteed answer remains A, B, C.

**Source-answer check:** CQ said A, B, C ✔. MD circled the same three hardware/tier combinations (GP standard, GP premium, BC memory-optimized), so it agrees ✔.
**Confidence:** Medium (D is now also feasible in major regions)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits

---

### Q30. 60-TB database: Azure SQL Database tier
<sub>ID U-036</sub>

**Sources:** MD-055   **Type:** MCQ   **Case study:** —

**Question:**
You have an on-premises Microsoft SQL Server 2022 instance that hosts a 60-TB production database named DB1. You plan to migrate DB1 to Azure. You need to recommend a hosting solution for DB1. Which Azure SQL Database service tier should you use to host DB1?

**Options:**
A. Hyperscale
B. Business Critical
C. General Purpose

**✅ Answer:** A

**Why:** The General Purpose and Business Critical tiers of Azure SQL Database top out at 4 TB of data. Hyperscale supports databases of up to 128 TB (the older limit was 100 TB), so it is the only Azure SQL Database tier that can host 60 TB.

**Source-answer check:** MD said A ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale?view=azuresql

---

### Q31. FileTable/FILESTREAM: choose Azure SQL offering
<sub>ID U-037</sub>

**Sources:** MD-019, CQ-036   **Type:** MCQ   **Case study:** —

**Question:**
You have an on-premises Microsoft SQL Server that uses the FileTables and Filestream features. You plan to migrate to Azure SQL. Which service should you use?

**Options:**
A. Azure SQL Database
B. SQL Server on an Azure Virtual Machine
C. Azure SQL Managed Instance
D. Azure Database for MySQL

**✅ Answer:** B

**Why:** FILESTREAM and FileTable aren't supported in Azure SQL Database or Azure SQL Managed Instance. The features comparison lists "No" for both. Only SQL Server on an Azure VM (full SQL Server with OS/file-system access) supports them. Azure Database for MySQL isn't SQL Server.

**Source-answer check:** MD & CQ said B ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/features-comparison

---

### Q32. Latest SQL version + SQL Agent, least admin
<sub>ID U-038</sub>

**Sources:** MD-035, CQ-052   **Type:** MCQ   **Case study:** —

**Question:**
You have a Microsoft SQL Server 2017 server. You need to migrate the server to Azure. The solution must meet the following requirements:
- Ensure that the latest version of SQL Server is used.
- Support the SQL Server Agent service.
- Minimize administrative effort.

What should you use?

**Options:**
A. an Azure SQL Database elastic pool
B. Azure SQL Database
C. SQL Server on Azure Virtual Machines
D. Azure SQL Managed Instance

**✅ Answer:** D

**Why:** SQL Managed Instance is a PaaS (evergreen/always-up-to-date) engine that Microsoft patches and upgrades, and it includes SQL Server Agent. Azure SQL Database (single database or elastic pool) has no SQL Server Agent and uses elastic jobs instead. A SQL Server VM supports Agent, but you must install, patch and upgrade it yourself, which means more administrative effort.

**Source-answer check:** MD & CQ said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/features-comparison

---

### Q33. Linked server to on-premises SQL2, least admin
<sub>ID U-039</sub>

**Sources:** MD-029, CQ-046, NOTE-013   **Type:** MCQ   **Case study:** —

**Question:**
You have two on-premises Microsoft SQL Server 2019 instances named SQL1 and SQL2. You need to migrate the databases hosted on SQL1 to Azure. The solution must meet the following requirements:
- The service that hosts the migrated databases must be able to communicate with SQL2 by using linked server connections.
- Administrative effort must be minimized.

What should you use to host the databases?

**Options:** (MD/CQ order; NOTE-013 lists the same options shuffled, with B = elastic pool, C = SQL VM, D = SQL MI)
A. a single Azure SQL database
B. SQL Server on Azure Virtual Machines
C. Azure SQL Managed Instance
D. an Azure SQL Database elastic pool

**✅ Answer:** C, Azure SQL Managed Instance (D in the NOTE-013 ordering)

**Why:** SQL Managed Instance supports linked servers to SQL Server and SQL Database targets (without distributed transactions) and is PaaS, so it needs the least administration. Azure SQL Database (single or pooled) doesn't support linked servers. A SQL Server VM supports linked servers but requires OS and SQL management.

**Source-answer check:** All sources agree on SQL Managed Instance ✔ (MD/CQ = C, NOTE = D due to option order).
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/features-comparison

---

### Q34. Merge replication: choose Azure SQL offering
<sub>ID U-040</sub>

**Sources:** MD-037, CQ-054   **Type:** MCQ   **Case study:** —

**Question:**
You have an on-premises Microsoft SQL Server 2019 database named SQL1 that uses merge replication. You need to migrate SQL1 to Azure. Which service should you use?

**Options:**
A. Azure SQL Edge
B. Azure SQL Database
C. SQL Server on Azure Virtual Machines
D. Azure SQL Managed Instance

**✅ Answer:** C

**Why:** Neither Azure SQL Database nor Azure SQL Managed Instance supports merge replication. Managed Instance supports transactional, snapshot and bidirectional replication, and SQL Database can only be a transactional or snapshot subscriber. Only full SQL Server on an Azure VM keeps merge replication. Azure SQL Edge is an IoT/edge engine (now retired) and isn't a migration target.

**Source-answer check:** MD & CQ said C ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/replication-to-sql-database

---

### Q35. SQL MI: lowest latency, highest memory per vCore
<sub>ID U-041</sub>

**Sources:** MD-034, CQ-051   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription. You need to deploy an Azure SQL managed instance that meets the following requirements:
- Optimize latency.
- Maximize the memory-to-vCore ratio.

Which service tier and hardware generation should you use?

**Options:**
Service tier: Business Critical | General Purpose | Hyperscale
Hardware generation: Premium-series | Standard-series (Gen 5) | Premium-series - memory optimized

**✅ Answer:** Service tier: Business Critical; Hardware generation: Premium-series - memory optimized

**Why:** Business Critical uses local SSD storage with about 1-2 ms I/O latency, compared with 5-10 ms for General Purpose and 3-5 ms for Next-gen General Purpose. Hyperscale isn't a SQL Managed Instance tier. Memory per vCore is 5.1 GB on standard-series, 7 GB on premium-series and 13.6 GB on memory-optimized premium-series, so memory-optimized premium-series gives the highest ratio.

**Source-answer check:** MD & CQ agree ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits

---

### Q36. SQL MI tier to minimize disk latency
<sub>ID U-042</sub>

**Sources:** MD-032, CQ-049   **Type:** MCQ   **Case study:** —

**Question:**
Your on-premises network contains a Microsoft SQL Server 2016 server that hosts a database named db1. You have an Azure subscription. You plan to migrate db1 to an Azure SQL managed instance. You need to create the SQL managed instance. The solution must minimize the disk latency of the instance. Which service tier should you use?

**Options:**
A. Business Critical
B. Hyperscale
C. General Purpose
D. Premium

**✅ Answer:** A

**Why:** SQL Managed Instance offers only General Purpose (remote storage, 5-10 ms; 3-5 ms on Next-gen GP) and Business Critical (local SSD, 1-2 ms). Hyperscale and Premium (DTU) are Azure SQL Database tiers, not Managed Instance tiers.

**Source-answer check:** MD & CQ said A ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits

---

### Q37. Seasonal peak: 24 vCores, >50k IOPS tier
<sub>ID U-043</sub>

**Sources:** MD-013, CQ-030   **Type:** MCQ   **Case study:** —

**Question:**
You are planning a solution that will use Azure SQL Database. Usage of the solution will peak from October 1 to January 1 each year. During peak usage, the database will require the following:
- 24 cores
- 500 GB of storage
- 124 GB of memory
- More than 50,000 IOPS

During periods of off-peak usage, the service tier of Azure SQL Database will be set to Standard. Which service tier should you use during peak usage?

**Options:**
A. Business Critical
B. Premium
C. Hyperscale

**✅ Answer:** A

**Why:** At 24 vCores (standard-series, about 124 GB memory), Business Critical delivers 4,000 IOPS per vCore, roughly 96,000 IOPS. General Purpose provides only 320 IOPS per vCore. Hyperscale could meet the numbers, but the database must go back to Standard every year: you can reverse-migrate out of Hyperscale only to General Purpose, and only within 45 days of the original migration. That rules out a recurring scale-down. Premium is a DTU tier and isn't sized in cores and memory. Business Critical can be scaled up and down freely.

**Source-answer check:** MD & CQ said A ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale?view=azuresql ; https://learn.microsoft.com/en-us/azure/azure-sql/database/reverse-migrate-from-hyperscale?view=azuresql

---

### Q38. Two projects: OS access vs CDC/auditing/reporting
<sub>ID U-044</sub>

**Sources:** MD-061   **Type:** Hotspot   **Case study:** —

**Question:**
You are working on two projects named Project1 and Project2 that will each migrate an on-premises app to Azure.

Project1 must meet the following requirements:
- Have access to the operating system of the database server that hosts the app.
- Have 25 TB of storage.

Project2 must meet the following requirements:
- Have change data capture (CDC) enabled.
- Support instance-level SQL auditing.
- Support managed backups.
- Have 10 TB of storage.
- Offload reports.

You need to recommend an Azure SQL solution for the projects. The solution must minimize costs. What should you include in the recommendation for each project?

**Options:**
Project1: Azure Kubernetes Service (AKS) | Azure SQL Database | Azure SQL Managed Instance in the Business Critical service tier | Azure SQL Managed Instance in the General Purpose service tier | SQL Server on Azure Virtual Machines
Project2: (same five choices)

**✅ Answer:** Project1: SQL Server on Azure Virtual Machines; Project2: Azure SQL Managed Instance in the Business Critical service tier

**Why:**
- **Project1:** Only SQL Server on Azure VMs gives access to the database server's operating system, and a VM can attach 25 TB.
- **Project2, why Managed Instance:** Instance-level SQL Server Audit and CDC point to Managed Instance (Azure SQL Database has no instance-level audit).
- **Project2, why Business Critical:** "Offload reports" requires a readable secondary. Only Business Critical includes a built-in read scale-out replica; General Purpose has none unless you pay for a second instance in a failover group.
- **Project2, storage:** Business Critical now supports 10 TB. Memory-optimized premium-series reaches 12-16 TB at 48 or more vCores, and premium-series reaches 16 TB at 64 or more vCores in major regions. So Business Critical is the cheapest single option that meets every requirement.

**Source-answer check:** MD said Project1 = SQL VM ✔; Project2 = MI General Purpose ✘. MD's reasoning ("BC only 4 TB") reflects the old standard-series limit, and General Purpose can't offload reports.
**Confidence:** Low. Disputed: the dump key picks General Purpose on storage/cost grounds, but current limits allow 10 TB on Business Critical, and only Business Critical satisfies "offload reports". No community answer found online.   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits ; https://learn.microsoft.com/en-us/azure/azure-sql/database/features-comparison

---

### Q39. Secure enclaves: vCore DC-series + Azure Attestation
<sub>ID U-045</sub>

**Sources:** MD-053   **Type:** Hotspot   **Case study:** —

**Question:**
You have an on-premises Microsoft SQL Server database named DB1. You have an Azure subscription. You need to migrate DB1 to an Azure SQL service that meets the following requirements:
- Protects the confidentiality of sensitive data from malware and high-privileged unauthorized database administrators
- Supports pattern matching for server-side database operations
- Uses a hardware-based encryption technology

Which Azure SQL service and attestation service should you include in the solution?

**Options:**
- Azure SQL service: Azure SQL Managed Instance | The Azure SQL Database DTU purchasing model | The Azure SQL Database vCore purchasing model
- Attestation service: Host Guardian Service (HGS) | Microsoft Azure Attestation | No attestation

**✅ Answer:** Azure SQL service: The Azure SQL Database vCore purchasing model; Attestation service: Microsoft Azure Attestation

**Why:** Always Encrypted with secure enclaves enables rich computations (pattern matching with LIKE) on encrypted columns. The hardware-based enclave is Intel SGX, which is available only with the DC-series hardware configuration in the vCore purchasing model of Azure SQL Database, and SGX attestation in Azure SQL Database is mandatory and uses Microsoft Azure Attestation. VBS enclaves (DTU or non-DC vCore) are software-based and support no attestation; HGS is for SQL Server on-premises/VMs. Managed Instance doesn't offer SGX enclaves. Note: Microsoft has announced the end of support for Intel SGX enclaves on 31 Oct 2027 (DC-series databases will be moved to standard-series with VBS). The exam answer stays vCore + Azure Attestation.

**Source-answer check:** MD said vCore + Microsoft Azure Attestation ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/always-encrypted-enclaves-plan ; https://learn.microsoft.com/en-us/azure/azure-sql/database/always-encrypted-enclaves-configure-attestation

---

### Q40. Single database autoscaling compute: service tier
<sub>ID U-046</sub>

**Sources:** MD-056   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription. You plan to provision a single Azure SQL database. You need to ensure that the database supports the autoscaling of compute resources. Which service tier should you choose?

**Options:**
A. Premium
B. General Purpose
C. Business Critical
D. Standard

**✅ Answer:** B

**Why:** Compute autoscaling is the serverless compute tier, which exists only in the vCore model for the General Purpose (and Hyperscale) service tiers. Premium and Standard are DTU tiers (no serverless); Business Critical supports provisioned compute only.

**Source-answer check:** MD said B ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql

---

### Q41. Pick platform for three projects
<sub>ID U-047</sub>

**Sources:** MD-070   **Type:** Hotspot   **Case study:** —

**Question:**
You have the projects shown in the following table.

| Name | Description |
|---|---|
| Project1 | A rapidly iterating cloud-based solution that integrates multiple software as a service (SaaS) solutions |
| Project2 | The migration of an existing on-premises Microsoft SQL Server 2019 database to Azure |
| Project3 | An app that supports multiple 20-TB databases |

You need to recommend an Azure database solution for each project. The solution must meet the following requirements:
- Project2 must minimize malware threats to the underlying operating system that hosts the migrated database.
- Administrative effort must be minimized where possible.

What should you recommend using for each project?

**Options:**
Project1 / Project2 / Project3 (same list each): Azure SQL Managed Instance | Azure SQL Database | Azure Cosmos DB | SQL Server on Azure VM

**✅ Answer:** Project1: Azure Cosmos DB; Project2: Azure SQL Managed Instance; Project3: Azure SQL Database (Hyperscale)

**Why:** Project1 – a rapidly iterating solution integrating many SaaS sources fits a schema-flexible, fully managed NoSQL store (Cosmos DB). Project2 – a PaaS target removes the customer-managed OS (and its malware exposure); Managed Instance gives near-100% SQL Server 2019 compatibility for a lift-and-shift. Project3 – Azure SQL Database Hyperscale supports databases up to 128 TB and is PaaS (lowest admin effort). Managed Instance tops out at 16 TB (32 TB per *instance* only in Next-gen GP), so "multiple 20-TB databases" doesn't fit; the MI limits page itself says to consider Hyperscale for larger storage. SQL Server on Azure VM works but adds OS/patching effort, violating "minimize administrative effort". The source key (SQL VM) reflects the old assumption that Azure SQL Database can't hold 20 TB.

**Second opinion:** Disputed: Project1 has no hard requirement. Cosmos DB (the source key) fits a rapidly changing schema across many SaaS feeds, while an independent review picked Azure SQL Database for a DP-300 context. Project2 (MI) and Project3 (Azure SQL Database Hyperscale, up to 128 TB) are firm.

**Source-answer check:** MD said P1 Cosmos DB ✔, P2 MI ✔, P3 SQL Server on Azure VM ✘ (Hyperscale holds up to 128 TB per database with less admin effort)
**Confidence:** Low (P1 wording is vague; P3 per current limits)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits?view=azuresql

---

### Q42. Web-app databases sharing compute, minimize cost
<sub>ID U-048</sub>

**Sources:** MD-074   **Type:** MCQ   **Case study:** —

**Question:**
You have five on-premises servers that have Microsoft SQL Server 2022 Enterprise installed. Each server contains multiple databases that store data for web apps and are backed up by using a third-party backup solution. You plan to migrate the databases to Azure. You need to recommend a solution to host the databases. The solution must meet the following requirements:
- Compute and storage resources must be shared across the databases.
- Costs must be minimized.

What should you include in the recommendation?

**Options:**
A. Azure SQL Managed Instance
B. Azure SQL Database
C. an Azure SQL Database elastic pool
D. SQL Server on Azure Virtual Machines

**✅ Answer:** C

**Why:** An elastic pool is the purpose-built, lowest-cost way to share a set of compute (vCores/eDTUs) and storage among many databases with varying usage – typical for web/SaaS app databases. Single databases (B) don't share resources; Managed Instance (A) shares instance resources but costs more; SQL Server on VMs (D) adds VM and license costs and admin effort. The third-party backup tool is a description of the current state, not a requirement to keep it.

**Source-answer check:** MD said D ✘ (keeping a third-party backup agent is not a stated requirement; the community vote on this question is ~93% for elastic pool)
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-pool-overview?view=azuresql (see also https://www.pass4success.com/microsoft/discussions/exam-dp-300-topic-4-question-132-discussion)

---

### Q43. Create partitioned table: object sequence
<sub>ID U-049</sub>

**Sources:** MD-042, CQ-059   **Type:** DragDrop   **Case study:** —

**Question:**
You have an Azure SQL database named DB1. You need to create a partitioned table in DB1. Which three objects should you create in sequence? To answer, move the appropriate objects from the list of objects to the answer area and arrange them in the correct order.

**Options:**
Objects: a partition scheme | an aligned index | a filegroup | a table | a partition function

**✅ Answer:** 1. a partition function → 2. a partition scheme → 3. a table

**Why:** Partitioning a table takes three steps: create the partition function (boundary values), create the partition scheme that maps partitions to filegroups (in Azure SQL Database only `ALL TO ([PRIMARY])` is possible), then create the table ON the partition scheme with the partitioning column. A filegroup can't be added in Azure SQL Database, and an aligned index is optional – it is not what makes "a partitioned table"; the goal is the table itself.

**Source-answer check:** CQ said function, scheme, table ✔; MD said function, scheme, aligned index ✘ (the table is the required third object; an aligned index is optional)
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/partitions/create-partitioned-tables-and-indexes

---

### Q44. ADatum: offload analytics for DB1/DB2
<sub>ID U-050</sub>

**Sources:** MD-003   **Type:** Hotspot   **Case study:** ADatum v1 (financial services)

**Question:**
(Case study: DB1 and DB2 on SVR1 – SQL Server 2016 Enterprise in AG1 – are migrated to an Azure SQL managed instance named Instance1. Availability requirement: "For DB1 and DB2, offload analytical workloads to a read-only database replica in the same Azure region." Business requirements: minimize costs and administrative effort.)
You need to recommend a service tier and a method to offload analytical workloads for the databases migrated from SVR1. The solution must meet the availability and business requirements. What should you recommend?

**Options:**
- Box 1 – Service tier: Business Critical | General Purpose | Premium
- Box 2 – Method: A failover group read-only listener | Geo-replicated secondary replicas | Read scale-out

**✅ Answer:** Box 1: Business Critical; Box 2: Read scale-out

**Why:** DB1/DB2 go to Instance1, a SQL Managed Instance. Managed Instance has only General Purpose (incl. Next-gen GP) and Business Critical tiers – "Premium" is an Azure SQL Database DTU tier and doesn't exist for MI. Only Business Critical includes a built-in readable secondary (read scale-out, included in the price) in the same region; General Purpose has no readable replica. A failover-group listener and geo-replicas put the readable copy in another region and cost extra.

**Source-answer check:** MD said Premium + Read scale-out — Box 2 ✔, Box 1 ✘ (Premium isn't an MI tier; read scale-out on MI needs Business Critical)
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits?view=azuresql ; https://learn.microsoft.com/en-us/azure/azure-sql/database/read-scale-out

---

### Q45. Contoso: PaaS prototype compute tier
<sub>ID U-051</sub>

**Sources:** MD-150   **Type:** MCQ   **Case study:** Contoso

**Question:**
(Case study: Contoso must minimize the compute costs of the PaaS solutions. The PaaS prototype compute-utilization exhibit (last 7 days, Max) shows near-0% use most of the time with short, intermittent spikes about once per day.)
Based on the PaaS prototype, which Azure SQL Database compute tier should you use?

**Options:**
A. Business Critical 4-vCore
B. Hyperscale
C. General Purpose v-vCore
D. Serverless

**✅ Answer:** D

**Why:** Long idle periods with brief, intermittent spikes are the textbook serverless pattern: compute auto-scales between min/max vCores, is billed per second, and (in General Purpose) can auto-pause during idle periods so only storage is billed – meeting "minimize compute costs". Provisioned Business Critical, Hyperscale or General Purpose bill for the full provisioned compute 24x7.

**Source-answer check:** MD said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql

---

### Q46. Auto-scale + per-second billing: serverless GP
<sub>ID U-052</sub>

**Sources:** MD-036, CQ-053   **Type:** Hotspot   **Case study:** —

**Question:**
You have a Microsoft SQL Server 2017 server that hosts five databases. You plan to migrate the databases to Azure. You need to recommend a solution that meets the following requirements:
- Automatically scales compute based on the workload demand
- Provides per-second billing

What should you include in the recommendation?

**Options:**
- Azure service: A single Azure SQL database in the provisioned compute tier | A single Azure SQL database in the serverless compute tier | An Azure SQL Database elastic pool | Azure SQL Managed Instance
- Service tier: Basic | General Purpose | Standard

**✅ Answer:** Azure service: A single Azure SQL database in the serverless compute tier; Service tier: General Purpose

**Why:** Serverless automatically scales compute with demand and bills compute per second. It's a vCore-model option available in General Purpose (and Hyperscale); Basic and Standard are DTU tiers that have no serverless option. Provisioned compute, elastic pools and Managed Instance are billed per hour for fixed provisioned resources.

**Source-answer check:** MD and CQ both said serverless + General Purpose ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql

---

### Q47. Dynamic CPU scaling + pause: serverless
<sub>ID U-053</sub>

**Sources:** MD-033, CQ-050   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription. You need to deploy an Azure SQL database. The solution must meet the following requirements:
- Dynamically scale CPU resources.
- Ensure that the database can be paused to reduce costs.

What should you use?

**Options:**
A. the Business Critical service tier
B. the serverless compute tier
C. an elastic pool
D. the General Purpose service tier

**✅ Answer:** B

**Why:** The serverless compute tier auto-scales vCores within the min/max range and supports auto-pause (General Purpose serverless), during which only storage is billed. A General Purpose provisioned database, Business Critical, or an elastic pool can't be paused.

**Source-answer check:** MD and CQ both said B ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql

---

### Q48. Hyperscale: add read-only replica, ApplicationIntent
<sub>ID U-054</sub>

**Sources:** NOTE-215   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription that contains the resources shown in the following table.

| Name | Type | Configuration |
|---|---|---|
| DB1 | Azure SQL Database | Hyperscale service tier; No secondary replicas |
| App1 | Azure Web Apps | App1 has read-only access to DB1. There are multiple instances of App1. |

You need to create a read-only replica of DB1 and configure the App1 instances to use the replica. What should you do?

**Options:**
- Box 1 – To add read-only replicas of DB1: Create a replica on the same logical server | Create a new logical server and configure geo-replication | Create a new logical server and configure an auto-failover group
- Box 2 – To configure App1 instances to access the read-only replica: Add an ApplicationIntent entry to the connection string | Add a MultiSubnetFailover entry to the App1 connection string | Create a dedicated endpoint and configure the App1 connection string to point to the endpoint

**✅ Answer:** Box 1: Create a replica on the same logical server; Box 2: Add an ApplicationIntent entry to the connection string

**Why:** Hyperscale high-availability (HA) secondary replicas share the primary's page servers, use the same logical server and database name, and are added just by setting the HA replica count (0-4) – no new server or data copy. Read-only connections are routed to them by `ApplicationIntent=ReadOnly` in the connection string. Geo-replication/failover groups need a second logical server and a full data copy (overkill for read scale-out). MultiSubnetFailover is for multi-subnet AG listeners; no "dedicated endpoint" is needed.

**Source-answer check:** NOTE hand-marked same logical server + ApplicationIntent ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale-replicas?view=azuresql

---

### Q49. Serverless exhibit: resume time and usage pattern
<sub>ID U-055</sub>

**Sources:** MD-011, CQ-028   **Type:** Hotspot   **Case study:** —

**Question:**
You are provisioning an Azure SQL database in the Azure portal as shown in the following exhibit.

Exhibit (Create SQL Database > Configure): Hardware configuration Gen5 (up to 40 vCores, 120 GB memory); **Max vCores: 6**; **Min vCores: 0.75** (2.25 GB min / 18 GB max memory); Auto-pause delay: **"Enable auto-pause" checkbox UNCHECKED** (Days 0, Hours 4, Minutes 0); Data max size 800 GB (240 GB log space allocated). Cost summary: General Purpose serverless (GP_S_Gen5), cost per GB 0.12 USD, estimated storage cost/month 119.60 USD, compute cost/vCore/second 0.000145 USD; "Serverless databases are billed in vCores based on a combination of CPU and memory utilization."

Use the drop-down menus to select the answer choice that completes each statement based on the information presented in the graphic.

**Options:**
- Statement 1: "After four hours of inactivity, the database requires [answer choice] to resume operations for new activities." – no extra time | up to 10 minutes | up to one minute
- Statement 2: "The database configuration reduces the cost of [answer choice] usage patterns." – intermittent and unpredictable | regular and high | steady and low

**✅ Answer:** Statement 1: no extra time; Statement 2: intermittent and unpredictable

**Why:** Auto-pause is disabled (checkbox cleared), so the database never pauses and there is no resume delay – the 4-hour value is ignored. (Had auto-pause been enabled, resume would take up to about a minute.) Serverless is price-optimized for intermittent, unpredictable usage with low average utilization; regular/high or steady use is better served by provisioned compute.

**Source-answer check:** MD and CQ both said no extra time + intermittent and unpredictable ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql

---

### Q50. SIM: db1 serverless 1–4 vCores, 45 GB
<sub>ID U-056</sub>

**Sources:** MD-052   **Type:** Simulation   **Case study:** —

**Question:**
SIMULATION. You need to ensure that db1 supports a minimum of one vCore, a maximum of four vCores, and a database of up to 45 GB. To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:**
(lab task – no options)

**✅ Answer:** Configure db1 as a **General Purpose serverless** database with Min vCores = 1, Max vCores = 4, Data max size = 45 GB.
Portal steps:
1. Azure portal → SQL databases → **db1** → **Settings > Compute + storage** (or Overview → Pricing tier link).
2. Service tier: **General Purpose** (vCore model); Compute tier: **Serverless**; Hardware: Standard-series (Gen5).
3. Set **Max vCores = 4** and **Min vCores = 1**.
4. Set **Data max size = 45 GB**. (Leave auto-pause as is – not part of the task.)
5. Select **Apply** and wait for the scale operation to finish.

PowerShell alternative:
```powershell
Set-AzSqlDatabase -ResourceGroupName <rg> -ServerName <server> -DatabaseName db1 `
  -Edition GeneralPurpose -ComputeModel Serverless -ComputeGeneration Gen5 `
  -MinVcore 1 -MaxVcore 4 -MaxSizeBytes 48318382080   # 45 GB
```

**Why:** A minimum *and* maximum vCore range exists only in the serverless compute tier (provisioned compute has a fixed vCore count). T-SQL `ALTER DATABASE ... MODIFY (SERVICE_OBJECTIVE = 'GP_S_Gen5_4', MAXSIZE = 45 GB)` sets max vCores and size but applies the default min vCores, so the portal/PowerShell/CLI is needed to set min vCores = 1. If db1 is in an elastic pool, it must be moved out first (serverless is for single databases only).

**Source-answer check:** MD set Max vCores 4 + 45 GB but omitted Min vCores = 1 / serverless ✘ (incomplete – min vCores must be set, which requires serverless)
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-create-configure?view=azuresql ; https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql

---

### Q51. SIM: db1 auto-pause after one hour
<sub>ID U-057</sub>

**Sources:** MD-046   **Type:** Simulation   **Case study:** —

**Question:**
SIMULATION. You need to configure db1 to pause automatically after one hour of inactivity. To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:**
(lab task – no options)

**✅ Answer:** Make db1 a **General Purpose serverless** single database with **auto-pause enabled, delay = 1 hour (60 minutes)**.
Portal steps:
1. Azure portal → SQL databases → **db1** → **Settings > Compute + storage**.
2. (If db1 is in an elastic pool, set **Elastic pool = No / None** first – serverless applies only to single databases.)
3. Service tier: **General Purpose**; Compute tier: **Serverless**.
4. Under **Auto-pause delay**, select **Enable auto-pause** and set **1 hour** (Days 0, Hours 1, Minutes 0).
5. Select **Apply**.

PowerShell alternative:
```powershell
Set-AzSqlDatabase -ResourceGroupName <rg> -ServerName <server> -DatabaseName db1 `
  -Edition GeneralPurpose -ComputeModel Serverless -ComputeGeneration Gen5 `
  -MaxVcore <n> -AutoPauseDelayInMinutes 60
```

**Why:** Only serverless databases can auto-pause (General Purpose GA; Hyperscale preview). The auto-pause delay is set in minutes (GP: 15 min – 7 days; -1 disables it); 60 minutes meets the requirement. Elastic pools have no serverless compute tier, so changing a pool's "compute tier" (as the source describes) isn't a valid method.

**Source-answer check:** MD: switch to serverless and set auto-pause delay to 1 hour ✔ in outcome, but its path "Elastic Pools > Compute tier > Serverless" ✘ (serverless is single-database only; change db1 itself)
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql ; https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-create-configure?view=azuresql

---

### Q52. Size elastic pool: three metrics
<sub>ID U-058</sub>

**Sources:** MD-004, CQ-001   **Type:** MultiSelect   **Case study:** —

**Question:**
You have 20 Azure SQL databases provisioned by using the vCore purchasing model. You plan to create an Azure SQL Database elastic pool and add the 20 databases. Which three metrics should you use to size the elastic pool to meet the demands of your workload? Each correct answer presents part of the solution. NOTE: Each correct selection is worth one point.

**Options:**
A. total size of all the databases
B. geo-replication support
C. number of concurrently peaking databases * peak CPU utilization per database
D. maximum number of concurrent sessions for all the databases
E. total number of databases * average CPU utilization per database

**✅ Answer:** A, C, E

**Why:** Microsoft's pool-sizing method: vCores = MAX(total number of DBs × average vCore utilization per DB, number of concurrently peaking DBs × peak vCore utilization per DB), and storage = sum of the data size of all databases in the pool. Geo-replication support and concurrent sessions are not sizing inputs.

**Source-answer check:** CQ said A, C, E ✔; MD said A, C, E ✔ (MD's option list was missing – options taken from CQ-001)
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-pool-overview?view=azuresql

---

### Q53. SQL VM: 4-TB volume with maximum IOPS
<sub>ID U-059</sub>

**Sources:** MD-179, NOTE-78   **Type:** MCQ   **Case study:** —

**Question:**
You have SQL Server on an Azure virtual machine. You need to add a 4-TB volume that meets the following requirements:
- Maximizes IOPs
- Uses premium solid state drives (SSDs)

What should you do?

**Options:**
A. Attach two mirrored 4-TB SSDs.
B. Attach a stripe set that contains four 1-TB SSDs.
C. Attach a RAID-5 array that contains five 1-TB SSDs.
D. Attach a single 4-TB SSD.

**✅ Answer:** B

**Why:** Premium SSD IOPS scale with disk size per disk, and striping (Storage Spaces) adds up the IOPS of every disk: four 1-TiB P30 disks give about 4 × 5,000 = 20,000 IOPS, versus 7,500 for a single 4-TiB P50. P30s also support read-only host caching (disks of 4 TiB or more don't). Mirroring gives no IOPS gain for writes and halves capacity; RAID-5 has a parity write penalty and isn't a recommended Azure layout.

**Source-answer check:** MD and NOTE both said B ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage?view=azuresql

---

### Q54. ARM template: SQL VM disk caching
<sub>ID U-060</sub>

**Sources:** MD-189   **Type:** Hotspot   **Case study:** —

**Question:**
You need to use an Azure Resource Manager (ARM) template to deploy an Azure virtual machine that will host a Microsoft SQL Server instance. The solution must maximize disk I/O performance for the SQL Server database and log files. How should you complete the template?

```json
"variables": {
  "dataDisks": {
    "caching": [Box 1], "dataDiskCount": 8, "logDisksCount": 1,
  ...
},
"resources": [
  ...
  { "type": "Microsoft.Compute/virtualMachines",
    ...
    "storageProfile": {
      "osDisk": { ... },
      "copy": [
        { "name": "dataDisks",
          "count": "[add(variables('dataDiskCount'), variables('logDisksCount'))]",
          "input": { "lun": "[copyIndex('dataDisks')]", "createOption": "empty",
            "caching": "[if(greaterOrEquals(copyIndex('dataDisks'), parameters('dataDiskCount')), [Box 2], variables('dataDisks').caching )]",
            "diskSizeGB": 1023,
  ...
```

**Options:**
- Box 1: None | ReadOnly | ReadWrite
- Box 2: None | ReadOnly | ReadWrite

**✅ Answer:** Box 1: ReadOnly; Box 2: None

**Why:** The copy loop creates 8 data disks (LUN 0-7) plus 1 log disk (LUN 8). The `if()` returns Box 2 when `copyIndex >= dataDiskCount` – i.e., only for the log disk – and otherwise uses `variables('dataDisks').caching` (Box 1) for the data disks. Microsoft's guidance: data-file disks = ReadOnly host caching; transaction-log disks = None; never ReadWrite on SQL data/log disks. So Box 1 (data) = ReadOnly and Box 2 (log) = None.

**Source-answer check:** MD said Box 1 None, Box 2 ReadOnly ✘ – its own explanation ("ReadOnly for data, None for log") is right, but it mapped them to the wrong boxes; per the template logic Box 1 is the data-disk setting and Box 2 is the log-disk setting
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage?view=azuresql

---

### Q55. Two P30 disks for 10,000 IOPS: object sequence
<sub>ID U-061</sub>

**Sources:** MD-188, NOTE-007-img065   **Type:** DragDrop   **Case study:** —

**Question:**
You are building an Azure virtual machine. You allocate two 1-TiB, P30 premium storage disks to the virtual machine. Each disk provides 5,000 IOPS. You plan to migrate an on-premises instance of Microsoft SQL Server to the virtual machine. The instance has a database that contains a 1.2-TiB data file. The database requires 10,000 IOPS. You need to configure storage for the virtual machine to support the database. Which three objects should you create in sequence? To answer, move the appropriate objects from the list of objects to the answer area and arrange them in the correct order.

**Options:**
- a virtual disk that uses the stripe layout
- a virtual disk that uses the mirror layout
- a volume
- a virtual disk that uses the simple layout
- a storage pool

**✅ Answer:** 1. a storage pool → 2. a virtual disk that uses the simple layout → 3. a volume

**Why:** The order (storage pool → virtual disk → volume) is certain. The data must be striped across both disks to get 2 TiB of capacity (> 1.2 TiB) and 2 × 5,000 = 10,000 IOPS; mirror gives only 1 TiB and no write-IOPS gain. In Windows Storage Spaces, the striping layout is named **Simple** ("stripes data across physical disks, maximizes capacity and increases throughput, no resiliency"); the only layouts are Simple, Mirror and Parity (`New-VirtualDisk -ResiliencySettingName Simple -NumberOfColumns 2`). There is no layout called "stripe" in Storage Spaces, so "simple layout" is the technically exact choice. Caveat: the classic dump key picks "stripe layout", reading it as plain-English striping; both describe the same RAID-0-style striping, so the exam's scored key may follow the dump.

**Source-answer check:** NOTE hand-marked simple layout ✔ (matches Storage Spaces terminology); MD said stripe layout ✘/disputed (same intent, but "stripe" isn't a Storage Spaces layout name)
**Confidence:** Low (both the stripe and simple options mean striping; which one the exam scores is unverified)   **Ref:** https://learn.microsoft.com/en-us/windows-server/storage/storage-spaces/deploy-standalone-storage-spaces ; https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage?view=azuresql

---

### Q56. Attach ultra disk to burstable SQL VM
<sub>ID U-062</sub>

**Sources:** MD-058   **Type:** DragDrop (ordering)   **Case study:** —

**Question:**
You have a burstable Azure virtual machine named VM1 that hosts an instance of Microsoft SQL Server. You need to attach an Azure ultra disk to VM1. The solution must minimize downtime on VM1. In which order should you perform the actions? To answer, move all actions from the list of actions to the answer area and arrange them in the correct order.

**Options:**
Actions: Resize VM1. | Set Enable Ultra disk compatibility to Yes. | Stop and deallocate VM1. | Attach the ultra disk. | Start VM1.

**✅ Answer:** 1. Stop and deallocate VM1 → 2. Resize VM1 → 3. Set Enable Ultra disk compatibility to Yes → 4. Start VM1 → 5. Attach the ultra disk

**Why:** Classic burstable B-series sizes aren't on the ultra-disk-compatible size list, so VM1 must be resized to a supported size. Do both changes in one deallocation: resize first (compatibility can't be enabled on an unsupported size), then enable Ultra disk compatibility, which requires the VM to be stopped/deallocated. Microsoft's CLI/PowerShell procedure is "deallocate → enable compatibility → start → attach": once compatibility is enabled, the ultra disk can be attached to the running VM, so attaching after the start keeps downtime as short as possible. (The portal walkthrough attaches while still deallocated and then starts the VM. That also works and is the dump's order, but it keeps VM1 offline for longer.)

**Source-answer check:** MD said Stop → Resize → Enable → Attach → Start ✘/disputed (valid, but attaching before the start doesn't minimize downtime; docs show attaching to the running VM after compatibility is enabled)
**Confidence:** Medium (the exam key may use the portal order Attach → Start)   **Ref:** https://learn.microsoft.com/en-us/azure/virtual-machines/disks-enable-ultra-ssd

---

### Q57. Custom-image SQL VM: automate patching and backups
<sub>ID U-063</sub>

**Sources:** MD-161, NOTE-10-img036   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure virtual machine based on a custom image named VM1. VM1 hosts an instance of Microsoft SQL Server 2019 Standard. You need to automate the maintenance of VM1 to meet the following requirements:
- Automate the patching of SQL Server and Windows Server.
- Automate full database backups and transaction log backups of the databases on VM1.
- Minimize administrative effort.

What should you do first?

**Options:**
A. Enable a system-assigned managed identity for VM1
B. Register the Azure subscription to the Microsoft.Sql resource provider (NOTE wording: "Register VM1 to the Microsoft.Sql resource provider")
C. Install an Azure virtual machine Desired State Configuration (DSC) extension on VM1
D. Register the Azure subscription to the Microsoft.SqlVirtualMachine resource provider (NOTE wording: "Register VM1 to the Microsoft.SqlVirtualMachine resource provider")

**✅ Answer:** D

**Why:** Automated Patching and Automated Backup are features of the SQL Server IaaS Agent extension. A VM built from a custom image (self-installed SQL Server) isn't registered automatically, so you must register it with the extension. The first step is registering the subscription with the Microsoft.SqlVirtualMachine resource provider; the SQL VM is then registered, creating the SQL virtual machine resource. Microsoft.Sql is the provider for Azure SQL Database/MI, not SQL on VMs. Neither a managed identity nor DSC provides these features.

**Source-answer check:** MD and NOTE both said D ✔ (option wording differs slightly – same answer)
**Confidence:** High   **Ref:** https://docs.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/sql-vm-resource-provider-register ; https://learn.microsoft.com/en-au/azure/azure-sql/virtual-machines/windows/sql-agent-extension-manually-register-single-vm?view=azuresql

---

### Q58. SQL VM disk: custom IOPS/throughput at minimal cost
<sub>ID U-064</sub>

**Sources:** MD-044, CQ-061   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription. You need to deploy an instance of SQL Server on Azure Virtual Machines. The solution must meet the following requirements:
- Custom performance configuration, such as IOPS, capacity, and throughput, must be supported.
- Costs must be minimized.

Which type of disk should you include in the solution?

**Options:**
A. Premium SSD v2
B. Premium SSD
C. Standard SSD
D. Ultra SSD

**✅ Answer:** A

**Why:** Only Premium SSD v2 and Ultra Disk let you set capacity, IOPS and throughput independently. Of the two, Premium SSD v2 is the lower-cost option (baseline 3,000 IOPS / 125 MBps included, then pay for extra), and Microsoft recommends it for SQL Server for best price-performance. Ultra Disk is the premium-priced choice for extreme IOPS or sub-millisecond latency. Premium SSD and Standard SSD have fixed performance tiers tied to disk size.

**Source-answer check:** MD said A ✔; CQ said D ✘ (Ultra also supports custom performance but costs more, so it fails "minimize costs")
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage?view=azuresql ; https://learn.microsoft.com/en-us/azure//virtual-machines/disks-types

---

### Q59. SQL VM disk types: TempDB and logs
<sub>ID U-065</sub>

**Sources:** CQ-042, MD-025   **Type:** DragDrop   **Case study:** —

**Question:**
You have an Azure subscription. You plan to deploy a new Azure virtual machine that will host a Microsoft SQL Server instance. You need to configure the disks on the virtual machine. The solution must meet the following requirements:
- Minimize latency for transaction logs.
- Minimize the impact on IO throughput of the virtual machine. *(MD wording: "Minimize VM allowed IO.")*

Which type of disk should you use for each workload? Each disk type may be used once, more than once, or not at all.

**Options:**
Disk types: Local | Premium SSD | Standard HDD | Standard SSD | Ultra Disk
Answer area: TempDB: ____ ; Transaction logs: ____

**✅ Answer:** TempDB: **Local** (local ephemeral SSD, D:\); Transaction logs: **Ultra Disk**

**Why:** The VM's uncached (remote disk) IOPS/throughput limit "applies only to I/O against data drives remotely attached to the VM, not the local I/O against the temp drive (D:\)", so putting tempdb on the local SSD keeps tempdb I/O from using up the VM's remote-disk I/O budget. Microsoft also recommends tempdb on the local temp disk for most workloads. Ultra Disk (or Premium SSD v2) is the choice for the log when you need sub-millisecond latency. A Premium SSD for tempdb would count against the VM's uncached I/O cap, which is the opposite of the second requirement.

**Source-answer check:** CQ said TempDB=Local, Logs=Ultra Disk ✔. MD said TempDB=Premium SSD ✘. MD's own explanation quotes the "place tempdb on the local ephemeral SSD" guidance and then picks Premium SSD, which contradicts it. Logs=Ultra Disk ✔ in both.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage

---

### Q60. SQL VM edition downgrade: uninstall first
<sub>ID U-066</sub>

**Sources:** MD-031, CQ-048   **Type:** MCQ   **Case study:** —

**Question:**
You have a SQL Server on Azure Virtual Machines instance named SQLVM1 that was deployed by using an Azure Marketplace SQL Server 2019 Enterprise image. You need to change the Microsoft SQL Server instance on SQLVM1 to the Standard edition. The solution must ensure licensing compliance. What should you do first?

**Options:**
A. From the SQL Server Installation Center on SQLVM1, run the Edition Upgrade wizard.
B. From SQLVM1, uninstall the SQL Server instance.
C. From the SQL Server Installation Center on SQLVM1, run the Repair wizard.
D. From the Azure portal, reconfigure SQLVM1.

**✅ Answer:** B

**Why:** Going from Enterprise to Standard is a downgrade. Microsoft's procedure is to back up all databases, completely uninstall SQL Server, reinstall it from Standard media, and then update the SQL VM edition property in the portal for billing. Edition Upgrade (A) only goes upward. Repair (C) doesn't change the edition. Changing the portal edition property (D) is the last step and is only valid once the installed edition matches. Before uninstalling you'd also back up the databases and remove the SQL IaaS Agent extension, but neither is offered as an option.

**Source-answer check:** All sources agree (B) ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/change-sql-server-edition

---

### Q61. Install SQL Server 2022 on RHEL VM
<sub>ID U-067</sub>

**Sources:** MD-069   **Type:** DragDrop   **Case study:** —

**Question:**
You have an Azure virtual machine named VM1 that runs Red Hat Enterprise Linux (RHEL). You need to install and configure Microsoft SQL Server 2022 Standard on VM1. Which three actions should you perform in sequence?

**Options:**
Actions:
- Run sudo yum localinstall mssql-server_2022.x86_64.rpm.
- Run sudo systemctl restart mssql-server.
- Run sudo /opt/mssql/bin/mssql-conf setup.
- Download the mssql-server-2022.repo file.
- Run sudo yum install -y mssql-server.

**✅ Answer:** 1 → Download the mssql-server-2022.repo file; 2 → Run `sudo yum install -y mssql-server`; 3 → Run `sudo /opt/mssql/bin/mssql-conf setup`

**Why:** This is the RHEL quickstart order. First download the repository config (`curl ... mssql-server-2022.repo` into /etc/yum.repos.d), then install the package with yum, then run mssql-conf setup to set the sa password and choose the edition (Standard). `yum localinstall` of an .rpm is the offline method and isn't needed once the repo is configured. A restart isn't required because mssql-conf setup starts the service.

**Source-answer check:** MD answer ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-red-hat

---

### Q62. Linux SQL IaaS Agent extension: OS and benefit
<sub>ID U-068</sub>

**Sources:** MD-057   **Type:** Hotspot   **Case study:** —

**Question:**
You plan to deploy an instance of SQL Server on Linux Azure Virtual Machines. The instance will run Microsoft SQL Server 2022 and use the SQL Server IaaS Agent extension for Linux. Which Linux operating system should you deploy, and which benefit will the SQL Server IaaS Agent extension provide?

**Options:**
Operating system: Red Hat Enterprise Linux (RHEL) | SUSE Linux Enterprise Server (SLES) | Ubuntu Linux
Benefit: Automatic backup | Automatic patching | Registration of a SQL virtual machine resource in Azure

**✅ Answer:** Operating system: **Ubuntu Linux**; Benefit: **Registration of a SQL virtual machine resource in Azure**

**Why:** When this question was written, the Linux extension's documented limitation was "only SQL Server VMs on Ubuntu Linux are supported; other Linux distributions are not." On Linux the extension runs only in lightweight mode, which offers license-type and edition management plus registration (it creates the separate *SQL virtual machine* resource). Automated backup and automated patching are Windows-only features of the extension. Note: the current Learn pages (updated Sept 2026) no longer list a distribution limitation, and Red Hat has announced RHEL support, so this question may be outdated. Ubuntu is the answer the exam key expects.

**Source-answer check:** MD said OS=RHEL ✘ (the exam-era docs supported only Ubuntu); Benefit=Registration ✔
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/linux/sql-server-iaas-agent-extension-linux ; https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/linux/sql-iaas-agent-extension-register-vm-linux

---

### Q63. Manage 10 SQL VMs with one account
<sub>ID U-069</sub>

**Sources:** MD-024, CQ-041   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription linked to a Microsoft Entra tenant (CQ: "Azure AD tenant"). The subscription contains 10 virtual machines that run Windows Server 2019 and host Microsoft SQL Server 2019 instances. You need to ensure that you can manage the SQL Server instances by using a single user account. What should you do first?

**Options:**
A. Enable a user-assigned managed identity on each virtual machine.
B. Deploy a Microsoft Entra Domain Services (Azure AD DS) domain and join the virtual machines to the domain.
C. Enable a system-assigned managed identity on each virtual machine.
D. Join the virtual machines to the Microsoft Entra tenant.

**✅ Answer:** B

**Why:** Microsoft Entra authentication for SQL Server needs SQL Server 2022 or later. SQL Server 2019 supports only Windows (AD) and SQL authentication. To use one account across all 10 instances, the VMs must be joined to an AD domain. Microsoft Entra Domain Services provides a managed domain whose accounts are synced from the tenant, so one domain account can be granted a Windows login on every instance. Managed identities (A, C) identify the VM itself, not an administrator. Entra-joining the VMs (D) doesn't give SQL Server 2019 a Windows domain principal it can use.

**Source-answer check:** All sources agree (B) ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/sql/relational-databases/security/authentication-access/azure-ad-authentication-sql-server-overview

---

### Q64. SQL VM series: IO and memory ratios
<sub>ID U-070</sub>

**Sources:** MD-221   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription. You need to deploy two virtual machines named VM1 and VM2 that will host Microsoft SQL Server instances. The solution must meet the following requirements:
- VM1 must provide the highest I/O throughput-to-vCore ratio.
- VM2 must provide the highest memory-to-vCore ratio.

Which series should you select for each virtual machine?

**Options:**
VM1: DCadsv5-series | Ebdsv5-series | Mv2-series
VM2: DCadsv5-series | Ebdsv5-series | Mv2-series

**✅ Answer:** VM1: **Ebdsv5-series**; VM2: **Mv2-series**

**Why:** Microsoft's VM-size guidance names Ebdsv5 as the high I/O throughput-to-vCore option for SQL Server (about 89 MB/s per vCore, alongside the Mbdsv3 series, which isn't offered here). The M-series family (Mv2 here, which goes up to about 12 TB) has the highest memory-to-vCore ratios, well above Ebdsv5's 8:1. DCadsv5 is a general-purpose confidential-computing series with a 4:1 ratio and low storage throughput, so it fits neither requirement.

**Source-answer check:** MD answer ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-vm-size

---

### Q65. Write Accelerator requires M-series VM
<sub>ID U-071</sub>

**Sources:** MD-039, CQ-056   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription. You plan to deploy an instance of SQL Server on Azure Virtual Machines that supports Write Accelerator. Which virtual machine series should you use?

**Options:**
A. E-series
B. G-series
C. H-series
D. M-series

**✅ Answer:** D

**Why:** "Write Accelerator is a disk feature that's available only for the M-Series VMs." It's used on the transaction-log drive (Premium Storage) for very low write latency. E, G and H series don't support it.

**Source-answer check:** All sources agree (D) ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/performance-guidelines-best-practices-storage

---


## D2 — Implement a secure environment (20–25%)

*65 questions*

### Q66. Four customer DBs: isolate access per database
<sub>ID U-072</sub>

**Sources:** MD-122   **Type:** MultiSelect   **Case study:** —

**Question:**
You have an Azure subscription. You create a logical SQL server that hosts four databases. Each database will be used by a separate customer. You need to ensure that each customer can access only its own database. The solution must minimize administrative effort. Which two actions should you perform? Each correct answer presents part of the solution.

**Options:** *(MD-122 has no option list. Options recovered from the Pass4Success DP-300 discussion, Topic 10 Q86. MD's explanation used different letters: its "B" = private endpoint and "C" = firewall rules.)*
A. Create a network security group (NSG).
B. Create a server-level firewall rule.
C. Create a private endpoint.
D. Create a database-level firewall rule.
E. Deny public access.

**✅ Answer:** C, D (create a private endpoint; create a database-level firewall rule)

**Why:** Database-level IP firewall rules are stored in, and apply to, a single database. Microsoft says to use them when "users of one database should be fully isolated from another database", whereas server-level rules (B) open every database on the server. The private endpoint gives customers private connectivity to the server, which is the second part of the published key. An NSG (A) can't filter per database, and Deny public access (E) on its own doesn't isolate databases. Caveat: firewall rules apply only to public-endpoint traffic, so in practice per-database isolation also depends on per-database (contained) users. This two-option pairing is the exam key, not a complete design.

**Source-answer check:** MD said "B and C" by its own lettering = private endpoint + firewall rules, the same content as C, D here ✔ (letter mapping differs). The Pass4Success community also picks C, D (about 95%).
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure

---

### Q67. 40 customer DBs: users + database firewall
<sub>ID U-073</sub>

**Sources:** CQ-069, MD-089   **Type:** MultiSelect   **Case study:** —

**Question:**
You have 40 Azure SQL databases, each for a different customer. All the databases reside on the same Azure SQL Database server. You need to ensure that each customer can only connect to and access their respective database. Which two actions should you perform? Each correct answer presents part of the solution.

**Options:** *(from CQ-069; MD-089 had no option list)*
A. Implement row-level security (RLS).
B. Create users in each database.
C. Configure the database firewall.
D. Configure the server firewall.
E. Create logins in the master database.
F. Implement Always Encrypted.

**✅ Answer:** B, C

**Why:** Contained users created in each customer's database (no server login) authenticate directly to that database and can connect only to it. Database-level IP firewall rules allow a customer's IP range into that one database, not the whole server. Server firewall rules (D) open every database. Logins in master (E) are server-level principals, which this requirement doesn't need and which would also let customers connect to master. RLS (A) and Always Encrypted (F) control rows or columns, not which database a customer can connect to.

**Source-answer check:** CQ said B, C ✔. MD said B, E ✘ (logins aren't needed; per-database isolation of connections comes from contained users plus database-level firewall rules).
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure

---

### Q68. Litware ResearchDB1: Always Encrypted setup sequence
<sub>ID U-074</sub>

**Sources:** MD-078, NOTE-131   **Type:** DragDrop   **Case study:** Litware

**Question:**
You create all of the tables and views for ResearchDB1. You need to implement security for ResearchDB1. The solution must meet the security and compliance requirements. Which three actions should you perform in sequence?
*(Relevant Litware requirements: store encryption keys in Azure Key Vault; encrypt the PII data at rest, in transit, and in use; use least privilege. ResearchApp1 populates and accesses ResearchDB1.)*

**Options:**
Actions:
- Run the Always Encrypted wizard.
- Create an Azure Key Vault instance and generate a secret.
- Create an Azure Key Vault instance and configure an access policy.
- Create a Microsoft Entra (Azure AD) managed identity.
- Register ResearchApp1 to Microsoft Entra ID (Azure AD).

**✅ Answer:** 1 → Register ResearchApp1 to Microsoft Entra ID; 2 → Create an Azure Key Vault instance and configure an access policy; 3 → Run the Always Encrypted wizard

**Why:** "Encrypt in use" means Always Encrypted, and keys must live in Key Vault. The client application decrypts data itself, so it needs an Entra identity (an app registration / service principal) that the vault's access policy can grant key permissions to (get, unwrapKey, wrapKey, sign, verify). This is the classic AE + AKV tutorial flow: register the app, create the vault and grant the app access, then run the Always Encrypted wizard to create the CMK in the vault and the CEK and encrypt the PII columns. "Generate a secret" is wrong because the CMK must be a Key Vault *key*, not a secret. A free-standing "managed identity" isn't something the app is registered with in this flow, and the option names no resource to attach it to. Registering ResearchApp1 is the key-holder identity step.

**Source-answer check:** MD said Register ResearchApp1 → Key Vault + access policy → wizard ✔. NOTE (hand-marked) said Create managed identity first ✘ (steps 2 and 3 match).
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-tutorial-getting-started (redirect target of always-encrypted-azure-key-vault-configure) ; https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine

---

### Q69. Column used for joins: CEK + deterministic
<sub>ID U-075</sub>

**Sources:** MD-088, CQ-068   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure SQL database named DB1 that contains two tables named Table1 and Table2. Both tables contain a column named Column1. Column1 is used for joins by an application named App1. You need to protect the contents of Column1 at rest, in transit, and in use. How should you protect the contents of Column1?

**Options:**
Encryption key: Column encryption key | Database encryption key | Service master key
Encryption type: Deterministic | Randomized | Transparent Data Encryption (TDE)

**✅ Answer:** Encryption key: **Column encryption key**; Encryption type: **Deterministic**

**Why:** Protecting data "in use" requires Always Encrypted, where a column encryption key (itself protected by a column master key) encrypts the column data. Deterministic encryption allows equality joins, point lookups, grouping and indexing. Randomized encryption doesn't support joins (without enclaves). TDE and the database encryption key protect data only at rest. The service master key is an instance-level key, not used for column protection.

**Source-answer check:** All sources agree ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine

---

### Q70. Comparisons on randomized encryption: enclaves
<sub>ID U-076</sub>

**Sources:** MD-117   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database named DB1. You need to encrypt DB1. The solution must meet the following requirements:
- Encrypt data in motion.
- Support comparison operators.
- Provide randomized encryption.

What should you include in the solution?

**Options:**
A. Always Encrypted with secure enclaves
B. Always Encrypted
C. column-level encryption
D. Transparent Data Encryption (TDE)

**✅ Answer:** A

**Why:** "To perform pattern matching by using comparison operators, sorting, and indexing on encrypted columns, adopt Always Encrypted with secure enclaves and apply randomized encryption." Plain Always Encrypted (B) supports no computation on randomized columns and only equality on deterministic ones. Column-level encryption with symmetric keys (C) and TDE (D) decrypt inside the engine and don't keep data encrypted in motion and in use in the same way.

**Source-answer check:** MD answer ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine

---

### Q71. Hide Salary from DBAs: AE key order
<sub>ID U-077</sub>

**Sources:** MD-087, CQ-067   **Type:** DragDrop   **Case study:** —

**Question:**
You have an Azure SQL database that contains a table named Employees. Employees contains a column named Salary. You need to encrypt the Salary column. The solution must prevent database administrators from reading the data in the Salary column and must provide the most secure encryption. Which three actions should you perform in sequence?

**Options:**
Actions:
- Encrypt the Salary column by using the randomized encryption type.
- Create a column encryption key.
- Enable Transparent Data Encryption (TDE).
- Encrypt the Salary column by using the deterministic encryption type.
- Apply a dynamic data mask to the Salary column.
- Create a column master key.

**✅ Answer:** 1 → Create a column master key; 2 → Create a column encryption key; 3 → Encrypt the Salary column by using the randomized encryption type

**Why:** Always Encrypted keeps keys out of the database engine, so DBAs see only ciphertext. The CEK is encrypted by the CMK, so the CMK (and its metadata) must exist first. Randomized encryption is more secure than deterministic, which leaks equality patterns. TDE only protects data at rest, and dynamic data masking doesn't stop privileged users, so neither stops DBAs.

**Source-answer check:** All sources agree ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine

---

### Q72. SIM: Always Encrypted LastName with cert store
<sub>ID U-078</sub>

**Sources:** MD-141   **Type:** Simulation (lab)   **Case study:** —

**Question:**
SIMULATION. You need to encrypt the LastName column of the SalesLT.Customer table in db1 by using Always Encrypted. You must use the Windows Certificate Store. You may need to use SQL Server Management Studio and the Azure portal.

**Options:** none (lab task)

**✅ Answer (steps, SSMS Always Encrypted wizard):**
1. In SSMS, connect to the logical server hosting **db1** (get the server name from the Azure portal if needed).
2. Object Explorer → Databases → **db1** → Tables → right-click **SalesLT.Customer** → **Encrypt Columns…** (or expand Columns and right-click **LastName** → Encrypt Column).
3. Introduction page → **Next**.
4. Column Selection: select **LastName**. Set Encryption Type to **Deterministic** (supports equality lookups; Randomized is also acceptable because the task doesn't specify a type). Leave Encryption Key = **CEK_Auto1 (New)** → **Next**.
5. Master Key Configuration: Select the key store provider = **Windows certificate store**, master key source = **Current User** (auto-generates the CMK certificate) → **Next**.
6. In-Place Encryption Settings (if shown) → **Next**. Run Settings: **Proceed to finish now** → **Next**.
7. Summary → **Finish** → wait for all tasks to succeed → **Close**.
8. (Verify) db1 → Security → Always Encrypted Keys shows CMK_Auto1 / CEK_Auto1, or run: SELECT name, encryption_type_desc FROM sys.columns WHERE encryption_type IS NOT NULL;

**Why:** The wizard creates the column master key (a certificate in the CurrentUser\My store), the column encryption key and its metadata, and then encrypts the column client-side, changing string collation to the required _BIN2. T-SQL alone can't do this because the engine never sees the keys. Manual alternative: Security → Always Encrypted Keys → New Column Master Key (Windows Certificate Store – Current User) → New Column Encryption Key → then run the wizard choosing that CEK.

**Source-answer check:** MD steps ✔ (same wizard flow)
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-tutorial-getting-started

---

### Q73. Data types Always Encrypted supports
<sub>ID U-079</sub>

**Sources:** MD-119   **Type:** MultiSelect   **Case study:** —

**Question:**
You have an Azure SQL database named DB1. DB1 has a table named Table1 that contains the following columns.

| Name | Type |
|---|---|
| Column1 | Ntext |
| Column2 | Geometry |
| Column3 | Image |
| Column4 | Varchar |
| Column5 | Datetime2 |

You plan to enable Always Encrypted for Table1. Which two columns support encryption? Each correct answer presents a complete solution.

**Options:** *(not in source; reconstructed. The choices are the five columns.)*
A. Column1
B. Column2
C. Column3
D. Column4
E. Column5

**✅ Answer:** Column4 (varchar) and Column5 (datetime2), i.e. D, E under the reconstructed lettering

**Why:** Always Encrypted isn't supported for columns of type xml, timestamp/rowversion, **image**, **ntext**, text, sql_variant, hierarchyid, geography, **geometry**, vector, alias or user-defined types. That rules out Column1 (ntext), Column2 (geometry) and Column3 (image). varchar can be encrypted (it needs a _BIN2 collation, which the wizard sets), and datetime2 can be encrypted.

**Source-answer check:** Source gave no answer or options; this answer is derived from the documentation.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine

---

### Q74. Contoso: audit destination for single dashboard
<sub>ID U-080</sub>

**Sources:** MD-151   **Type:** MCQ   **Case study:** Contoso

**Question:**
Which audit log destination should you use to meet the monitoring requirements?
*(Contoso monitoring requirement: "Use a single dashboard to review security and audit data for all the PaaS databases.")*

**Options:**
A. Azure Storage
B. Azure Event Hubs
C. Azure Log Analytics

**✅ Answer:** C

**Why:** Sending audit logs from every PaaS database to one Log Analytics workspace lets you query and visualise them together (Azure Monitor workbooks/dashboards, Defender for Cloud), which gives you a single place to review security and audit data. Azure Storage holds .xel files per server or database that you have to read with SSMS or sys.fn_get_audit_file, so it gives no dashboard. Event Hubs only streams events to an external consumer (such as a SIEM) and doesn't store them for review.

**Source-answer check:** MD answer ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/auditing-overview

---

### Q75. SQL VM: audit SELECTs on CustomerPII
<sub>ID U-081</sub>

**Sources:** MD-160   **Type:** MultiSelect   **Case study:** —

**Question:**
You have SQL Server on an Azure virtual machine that contains a database named DB1. DB1 contains a table named CustomerPII. You need to record whenever users query the CustomerPII table. Which two options should you enable? Each correct answer presents part of the solution.

**Options:** *(not in MD-160; recovered from the CloudThat DP-300 sample-question page)*
A. SQL Server audit
B. database audit specification
C. server audit specification
D. a server principal

**✅ Answer:** A, B (a SQL Server audit object plus a database audit specification)

**Why:** In SQL Server Audit, the *SQL Server Audit* object is created at instance level and defines the target (file or event log). A *Database Audit Specification* belongs to that audit and collects database-level actions, such as a `SELECT` action on `dbo.CustomerPII`. A server audit specification (C) only holds server-level action groups and can't target a single table. A server principal (D) isn't an audit component. A specification can't record anything without the audit object, so A is required.

**Source-answer check:** MD gave no letters; its explanation implies audit object + database audit specification ✔. Note: CloudThat's published key is "b and c" ✘. A server audit specification can't target a table, and the database audit specification needs the SQL Server Audit object as its target.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/auditing/sql-server-audit-database-engine

---

### Q76. Track reads of confidential column, 365 days
<sub>ID U-082</sub>

**Sources:** CQ-063, MD-083   **Type:** MultiSelect   **Case study:** —

**Question:**
You have a new Azure SQL database. The database contains a column that stores confidential information. You need to track each time values from the column are returned in a query. The tracking information must be stored for 365 days from the date the query was executed. Which three actions should you perform? Each correct answer presents part of the solution.

**Options:** *(CQ-063 version. MD-083 had no options.)*
A. Turn on auditing and write audit logs to an Azure Storage account.
B. Add extended properties to the column.
C. Turn on auditing and write audit logs to an Event Hub.
D. Apply sensitivity labels named Highly Confidential to the column.
E. Turn on Azure Defender for SQL.

**✅ Answer:** A, D, E

**Why:** A sensitivity label on the column (D) makes SQL auditing fill the `data_sensitivity_information` field, which records the labels of data a query returned. Auditing to a Storage account (A) with retention set to 365 days stores the records for the required year. An Event Hub (C) only streams events and doesn't retain them for 365 days, so it doesn't meet the storage requirement. In the original version of this item the third correct option was "Turn on Advanced Data Security", which was renamed Azure Defender for SQL / Microsoft Defender for SQL. That option is E in this version (Pass4Success key A, C, D on the original lettering, about 97% community agreement). Today Data Discovery & Classification is built in without Defender, so E is a legacy-key choice.

**Source-answer check:** CQ said A, C, D ✘. CQ appears to have kept the original letters after the options were reshuffled, so its "C" is now Event Hub, which can't meet the 365-day storage requirement. The correct mapping is A, D, E. MD gave no answer.
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/data-discovery-and-classification-overview ; https://learn.microsoft.com/en-us/azure/azure-sql/database/auditing-overview

---

### Q77. ADatum: App1 Kerberos access to Instance1
<sub>ID U-083</sub>

**Sources:** MD-076   **Type:** DragDrop   **Case study:** ADatum v1 (financial)

**Question:**
You need to recommend an authentication solution for App1 access to DB1 and DB2 after their migration to Instance1. The solution must meet the availability requirements. Which actions should you perform in sequence? Each action may be used once, more than once, or not at all.
*(Case context: App1 runs on Microsoft Entra hybrid joined Windows Server 2022 servers and uses Kerberos authentication. The on-prem corp.adatum.com AD domain **already syncs** with the adatum.com Entra tenant. Instance1 is an Azure SQL managed instance. Availability requirement: "After the migration, App1 must maintain access to DB1 and DB2.")*

**Options:**
Answer area: Step 1 (pre-filled): Enable Microsoft Entra authentication on Instance1. Step 2: ____ Step 3: ____
Actions:
- Enable a system-assigned managed identity.
- Enable a system-assigned service principal.
- Grant admin consent to an app registration in Microsoft Entra.
- Implement Microsoft Entra Cloud Sync.

**✅ Answer:** Step 2: **Enable a system-assigned service principal**; Step 3: **Grant admin consent to an app registration in Microsoft Entra**

**Why:** Windows Authentication (Kerberos) for Entra principals on SQL MI requires Entra authentication on the instance (Step 1). Microsoft's MI configuration page then says: "you need to enable a system-assigned service principal on each SQL managed instance… You also need to grant admin consent to each service principal". Consent is granted via Entra ID → App registrations → `<instance> principal` → API permissions → Grant admin consent. AD-to-Entra sync is a one-time prerequisite "if this hasn't already been done", and the case says corp.adatum.com already syncs, so implementing Cloud Sync adds nothing. The system-assigned managed identity is the MI's primary identity for Graph or Key Vault access, not the Kerberos service principal.

**Source-answer check:** MD said Step 2 = Cloud Sync, Step 3 = system-assigned service principal ✘. Sync already exists in the case, and MD's own explanation wrongly says admin consent isn't needed, which contradicts the MI docs.
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/winauth-azuread-kerberos-managed-instance ; https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/winauth-azuread-setup

---

### Q78. Litware SERVER1: Entra authentication sequence
<sub>ID U-084</sub>

**Sources:** MD-079, DEMO-2, NOTE-296   **Type:** DragDrop   **Case study:** Litware

**Question:**
You need to configure user authentication for the SERVER1 databases. The solution must meet the security and compliance requirements. Which three actions should you perform in sequence?
*(Litware: SERVER1 databases migrate to Azure SQL Database. Requirement: "Authenticate database users by using Active Directory credentials" (hybrid Entra ID, litwareinc.com). Least privilege.)*

**Options:**
Actions:
- Create a user in the master database
- Modify the Azure SQL server administrator account
- Create contained database users
- Create a Microsoft Entra (Azure AD) administrator for the logical server
- Connect to the databases by using a Microsoft Entra (Azure AD) account
- Enable the contained database authentication option

**✅ Answer:** 1 → Create a Microsoft Entra administrator for the logical server; 2 → Create contained database users; 3 → Connect to the databases by using a Microsoft Entra account

**Why:** This follows Microsoft's documented sequence: assign a Microsoft Entra administrator to the server → create SQL principals (contained users, `CREATE USER … FROM EXTERNAL PROVIDER`) mapped to Entra identities → connect to the database with Entra identities. Contained users avoid master logins (least privilege). "Enable the contained database authentication option" is an `sp_configure` setting for boxed SQL Server and isn't needed or available in Azure SQL Database, where user databases are already contained.
Dispute: MD orders it admin → connect → create users, arguing that the Entra admin must connect before creating users. That is true in practice, but the item's "Connect…" step means users connecting with their Entra accounts, which matches the documented final step.

**Source-answer check:** DEMO-2 ✔. MD put "connect" before "create users" ✘ (it reads "connect" as the admin's connection; the documented order puts connect last). NOTE put "Enable the contained database authentication option" second ✘ (not an Azure SQL Database step).
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-overview

---

### Q79. App1 SQL auth access to DB2 only
<sub>ID U-085</sub>

**Sources:** MD-098, CQ-078, NOTE-162   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription that contains a server named Server1. Server1 hosts two Azure SQL databases named DB1 and DB2.
You plan to deploy a Windows app named App1 that will authenticate to DB2 by using SQL authentication.
You need to ensure that App1 can access DB2. The solution must meet the following requirements:
- App1 must be able to view only DB2.
- Administrative effort must be minimized.

What should you create?

**Options:**
A. a contained database user for App1 on DB2
B. a login for App1 on Server1
C. a contained database user from an external provider for App1 on DB2
D. a contained database user from a Windows login for App1 on DB2

**✅ Answer:** A

**Why:** A contained database user with a password (`CREATE USER App1 WITH PASSWORD = '...'` in DB2) authenticates at the database level with SQL authentication, so App1 can see only DB2 and nothing has to be created in master. A login on Server1 (B) needs a login plus a mapped user, which is more work and is a server-level principal. FROM EXTERNAL PROVIDER (C) is for Microsoft Entra identities, not SQL authentication. Windows logins (D) aren't supported in Azure SQL Database.

**Source-answer check:** All sources say A ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/logins-create-manage

---

### Q80. Reusable SQL login for App1 (db_datareader)
<sub>ID U-086</sub>

**Sources:** MD-090, CQ-070, NOTE-22   **Type:** DragDrop   **Case study:** —

**Question:**
You have an Azure SQL Database instance named DatabaseA on a server named Server1.
You plan to add a new user named App1 to DatabaseA and grant App1 db_datareader permissions. App1 will use SQL Server Authentication.
You need to create App1. The solution must ensure that App1 can be given access to other databases by using the same credentials.
Which three actions should you perform in sequence? To answer, move the appropriate actions from the list of actions to the answer area and arrange them in the correct order.

**Options:**
Actions:
- On the master database, run `CREATE LOGIN [App1] FROM EXTERNAL PROVIDER;`
- On DatabaseA, run `CREATE USER [App1] WITH PASSWORD = 'P@ssW0rd!';`
- On DatabaseA, run `ALTER ROLE db_datareader ADD MEMBER [App1];`
- On the master database, run `CREATE LOGIN [App1] WITH PASSWORD = 'P@ssW0rd!';`
- On DatabaseA, run `CREATE USER [App1] FROM LOGIN [App1];`

(The password reads `P@aaW0rd!` in the MD/CQ copies and `P@ssW0rd!` in the NOTE copy. This is only a transcription difference and doesn't change the answer.)

**✅ Answer:** 1. On master: `CREATE LOGIN [App1] WITH PASSWORD = '...';` → 2. On DatabaseA: `CREATE USER [App1] FROM LOGIN [App1];` → 3. On DatabaseA: `ALTER ROLE db_datareader ADD MEMBER [App1];`

**Why:** A SQL authentication login in master holds one set of credentials that can be mapped to a user in any database on the server, so App1 can reuse them. The user is then created from that login in DatabaseA and added to db_datareader. A contained user WITH PASSWORD exists in only one database, so its credentials can't be reused. FROM EXTERNAL PROVIDER is Microsoft Entra authentication, not SQL authentication.

**Source-answer check:** All sources agree ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/logins-create-manage

---

### Q81. Managed identity database user WITH OBJECT_ID
<sub>ID U-087</sub>

**Sources:** MD-145   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription that contains an Azure SQL database named DB1 and a managed identity.
You need to create a user in DB1 that will be used by the managed identity to perform automated tasks.
How should you complete the query? To answer, select the appropriate options in the answer area.
```sql
CREATE USER "AutomationAccount"
FROM [Box 1]
WITH [Box 2] = '1133a5a2-4c19-47d7-c2d5-42b289997918f'
EXEC sp_addrolemember 'db_owner', "AutomationAccount"
```

**Options:**
Box 1: ASYMMETRIC KEY | EXTERNAL PROVIDER | FOR CERTIFICATE
Box 2: KEY | LOGIN | OBJECT_ID

**✅ Answer:** Box 1: EXTERNAL PROVIDER; Box 2: OBJECT_ID

**Why:** Managed identities are Microsoft Entra service principals, so the user is created with `FROM EXTERNAL PROVIDER`. The `WITH OBJECT_ID = '<Entra object ID>'` extension binds the user to that exact identity, which matters when display names aren't unique, and lets you choose your own user name ("AutomationAccount"). ASYMMETRIC KEY and CERTIFICATE users are for module signing, and LOGIN isn't a valid WITH option here.

**Source-answer check:** MD said EXTERNAL PROVIDER / OBJECT_ID ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity ; https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure

---

### Q82. Contained Entra user in new DB1
<sub>ID U-088</sub>

**Sources:** MD-085, CQ-065   **Type:** DragDrop   **Case study:** —

**Question:**
You have a new Azure SQL database named DB1 on an Azure SQL server named AzSQL1. The only user who was created is the server administrator.
You need to create a contained database user in DB1 who will use Microsoft Entra ID (Azure AD) for authentication.
Which three actions should you perform in sequence? To answer, move the appropriate actions from the list of actions to the answer area and arrange them in the correct order.

**Options:**
Actions:
- Create a login in the master database.
- From the Azure portal, assign the SQL DB Contributor role to the user.
- Connect to DB1 by using the server administrator account.
- Set the Microsoft Entra (Active Directory) Admin for AzSQL1.
- Connect to DB1 by using the Microsoft Entra (Active Directory) admin account.
- Create a user by using the FROM EXTERNAL PROVIDER clause.

**✅ Answer:** 1. Set the Microsoft Entra admin for AzSQL1 → 2. Connect to DB1 by using the Microsoft Entra admin account → 3. Create a user by using the FROM EXTERNAL PROVIDER clause.

**Why:** Microsoft Entra authentication is turned on by setting the Entra admin. `CREATE USER ... FROM EXTERNAL PROVIDER` must be run by a principal that is signed in with a Microsoft Entra identity. Microsoft Learn says: "To create contained users mapped to Microsoft Entra identities, you must be logged in by using a Microsoft Entra account." The SQL authentication server admin can't create Entra users. A login in master isn't needed for a contained user, and SQL DB Contributor is an Azure control-plane role that gives no data access.

**Source-answer check:** MD & CQ said step 2 = "Connect to DB1 by using the server administrator account" ✘. That's wrong because a SQL-auth server admin can't run FROM EXTERNAL PROVIDER (the explanation text in both sources even says to sign in with an Entra identity). Steps 1 and 3 agree ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/logins-create-manage ; https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure

---

### Q83. Entra User1 cannot connect to logical server
<sub>ID U-089</sub>

**Sources:** MD-099, CQ-079   **Type:** MCQ   **Case study:** —

**Question:**
You create five Azure SQL Database instances on the same logical server. In each database, you create a user for a Microsoft Entra (Azure AD) user named User1.
User1 attempts to connect to the logical server by using Azure Data Studio and receives a login error.
You need to ensure that when User1 connects to the logical server by using Azure Data Studio, User1 can see all the databases.
What should you do?

**Options:**
A. Create User1 in the master database.
B. Assign User1 the db_datareader role for the master database.
C. Assign User1 the db_datareader role for the databases that User1 creates.
D. Grant SELECT on sys.databases to public in the master database.

**✅ Answer:** A

**Why:** User1 exists only as contained users in the five user databases. A connection to the logical server that doesn't name a database goes to master, where User1 has no principal, so the login fails. Creating User1 in master (`CREATE USER [User1] FROM EXTERNAL PROVIDER`, or a login plus a user) lets the connection succeed, and User1 can then list the databases. B and D can't be granted to a principal that doesn't exist in master, and C doesn't fix the connection to master.

**Source-answer check:** All sources say A ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/logins-create-manage

---

### Q84. Web app managed identity database user
<sub>ID U-090</sub>

**Sources:** MD-109, NOTE-10   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription that is linked to a Microsoft Entra (Azure AD) tenant named contoso.com. The subscription contains an Azure SQL database named SQL1 and an Azure web app named app1. App1 has the managed identity feature enabled.
You need to create a new database user for app1.
How should you complete the Transact-SQL statement? To answer, select the appropriate options in the answer area.
```sql
CREATE USER [Box 1] FROM [Box 2]
```

**Options:**
Box 1: [App1] | [Contoso\app1] | [App1@contoso.com]
Box 2: login | Windows | EXTERNAL PROVIDER

**✅ Answer:** Box 1: [App1]; Box 2: EXTERNAL PROVIDER

**Why:** A system-assigned managed identity is a service principal whose display name is the same as the resource (the web app) name, so the user is created as `CREATE USER [App1] FROM EXTERNAL PROVIDER;`. The UPN form (`App1@contoso.com`) is used only for Entra *users*. The `DOMAIN\name` form and FROM WINDOWS are for on-premises Windows logins, which Azure SQL Database doesn't support. FROM LOGIN would need an existing login.

**Source-answer check:** MD & NOTE agree ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure

---

### Q85. Service principal app1 access to db1
<sub>ID U-091</sub>

**Sources:** MD-114   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription that contains a logical SQL server. The server hosts two databases named db1 and db2 and a Microsoft Entra service principal named app1.
You need to ensure that app1 can access db1. The solution must use the principle of least privilege.
How should you complete the Transact-SQL statement? To answer, select the appropriate options in the answer area.
```sql
CREATE [Box 1] [app1] [Box 2]
```

**Options:**
Box 1: CREDENTIAL | LOGIN | USER
Box 2: FOR LOGIN app1 | FROM EXTERNAL PROVIDER | FROM LOGIN app1 | WITHOUT LOGIN

**✅ Answer:** Box 1: USER; Box 2: FROM EXTERNAL PROVIDER

**Why:** Running `CREATE USER [app1] FROM EXTERNAL PROVIDER;` in db1 creates a contained database user for the service principal, scoped to db1 only, which is least privilege. A LOGIN is a server-level principal and would also need a mapped user. FROM/FOR LOGIN need a login that doesn't exist. WITHOUT LOGIN creates a user that can't authenticate. CREDENTIAL isn't a principal.

**Source-answer check:** MD said USER / FROM EXTERNAL PROVIDER ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-service-principal

---

### Q86. SIM: add Entra user2 read-only to db1
<sub>ID U-092</sub>

**Sources:** MD-051   **Type:** Simulation   **Case study:** —

**Question:**
SIMULATION. You need to add a Microsoft Entra user named user2-12345678@examusers.com to db1. User2-12345678 must be able to read data from all the tables in db1 without being able to modify the data.
To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:**
(lab task: no options)

**✅ Answer (steps):**
1. (If not already set) Azure portal > SQL servers > the server that hosts db1 > **Settings > Microsoft Entra ID** > **Set admin** > select your lab account > **Select** > **Save**.
2. In SSMS, connect to the server with **Microsoft Entra MFA / Entra Integrated** as the Entra admin. On the **Connection Properties** tab, set the database to **db1**.
3. Open a new query window on db1 and run:
```sql
CREATE USER [user2-12345678@examusers.com] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [user2-12345678@examusers.com];
```
4. Optional check: `SELECT name, type_desc FROM sys.database_principals WHERE name LIKE 'user2%';`

**Why:** An Entra user must be created with FROM EXTERNAL PROVIDER while you are signed in with an Entra identity. db_datareader grants SELECT on all tables and views and no write permissions. The source's SSMS dialog route ("Windows user" type, then per-table SELECT on the Securables page) also gives read-only access but is error-prone. "Windows user" isn't a valid principal type in Azure SQL Database, and per-table grants don't cover tables added later.

**Source-answer check:** MD's GUI method (Windows user + Securables SELECT) is partly wrong ✘. Its T-SQL equivalent (FROM EXTERNAL PROVIDER + db_datareader) matches ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure ; https://learn.microsoft.com/en-us/azure/azure-sql/database/logins-create-manage

---

### Q87. SIM: let runbook create Entra users
<sub>ID U-093</sub>

**Sources:** MD-048   **Type:** Simulation   **Case study:** —

**Question:**
SIMULATION. You plan to create an automation runbook that will create database users in db1 from Microsoft Entra identities.
You need to configure sql12345678 to support the creation of new database users.
To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:**
(lab task: no options)

**✅ Answer (steps):** A runbook authenticates as a service principal or managed identity, which can't use delegated Graph lookups. So the **server identity** of sql12345678 must exist and be allowed to read Microsoft Entra ID.
1. Azure portal: check that the correct directory is selected (**Directories + subscriptions**).
2. **SQL servers > sql12345678 > Security > Identity** > set **System assigned managed identity** to **On** > **Save**. Alternatively, use PowerShell: `Set-AzSqlServer -ResourceGroupName <rg> -ServerName sql12345678 -AssignIdentity`.
3. Give that server identity permission to read Entra ID: add it to the Microsoft Entra **Directory Readers** role (Entra ID > Roles and administrators > Directory Readers > Add assignments > select the sql12345678 identity), or assign the Graph app permissions User.Read.All, GroupMember.Read.All and Application.Read.All with PowerShell. You need the Privileged Role Administrator role for this.
4. Make sure a **Microsoft Entra admin** is set on sql12345678 (**Settings > Microsoft Entra ID > Set admin > Save**). The runbook's identity can be that admin or can be granted ALTER ANY USER in db1.

**Why:** Microsoft Learn says that for a service principal to run `CREATE USER ... FROM EXTERNAL PROVIDER`, "the SQL engine tries to use its server identity… The server identity must exist and have the Microsoft Graph query permissions or the operations fail" (error 33134 "Server identity is not configured" otherwise). The source's steps (the "Grant permissions" banner on the Entra admin page) are written for SQL Managed Instance. For a logical server you enable the identity and assign Directory Readers or the Graph permissions yourself.

**Source-answer check:** MD gave the MI portal-banner steps (grant read permission + set Entra admin). The concept matches ✔, but the steps are for the wrong product. For a logical server, the identity must be enabled and the role assigned explicitly.
**Confidence:** Medium (the lab grader's exact expectation is unknown. Setting the Entra admin plus server identity + Directory Readers covers both interpretations.)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-service-principal ; https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity

---

### Q88. SIM: set your account as Entra admin
<sub>ID U-094</sub>

**Sources:** MD-127   **Type:** Simulation   **Case study:** —

**Question:**
SIMULATION. You need to configure your user account as the Microsoft Entra admin for the server named sql12345678.
To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:**
(lab task: no options)

**✅ Answer (steps):**
1. Azure portal: in **Directories + subscriptions**, select the directory that contains sql12345678.
2. Search **SQL servers** > open **sql12345678**.
3. **Settings > Microsoft Entra ID** > **Set admin**.
4. Search for your lab user account, select it > **Select**.
5. Click **Save** and wait for the operation to finish (the admin then appears in the Microsoft Entra admin field).
PowerShell alternative: `Set-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName <rg> -ServerName sql12345678 -DisplayName "<your UPN>"`. CLI alternative: `az sql server ad-admin create --resource-group <rg> --server-name sql12345678 --display-name <UPN> --object-id <id>`.

**Why:** This is the documented procedure for a logical server. The source's "grant read permissions" banner applies to SQL Managed Instance, and its "Azure AD B2C role-assignment" alternative isn't relevant.

**Source-answer check:** MD's main steps (Set admin > select > Save) ✔. Its MI-specific and B2C extras are not needed ✘.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure

---

### Q89. Let user1 create SP logins in SQLMI1
<sub>ID U-095</sub>

**Sources:** NOTE-038   **Type:** DragDrop   **Case study:** —

**Question:**
You have a Microsoft Entra (Azure AD) tenant named contoso.com that contains a user named user1@contoso.com and an Azure SQL managed instance named SQLMI1.
You need to ensure that user1@contoso.com can create logins in SQLMI1 that map to Microsoft Entra (Azure AD) service principals.
Which three actions should you perform in sequence? To answer, move the appropriate actions from the list of actions to the answer area and arrange them in the correct order.

**Options:**
Actions:
- Run `CREATE LOGIN [user1@contoso.com] FROM EXTERNAL PROVIDER` on the master database.
- Run `ALTER SERVER ROLE securityadmin ADD MEMBER [user1@contoso.com]`.
- Create a managed identity for SQLMI1.
- Grant SQLMI1 read access to Microsoft Entra ID (Azure AD).
- Run `CREATE USER [user1@contoso.com] FROM LOGIN [user1@contoso.com]`.

**✅ Answer:** 1. Grant SQLMI1 read access to Microsoft Entra ID (Directory Readers) → 2. Run `CREATE LOGIN [user1@contoso.com] FROM EXTERNAL PROVIDER` on master → 3. Run `ALTER SERVER ROLE securityadmin ADD MEMBER [user1@contoso.com]`.

**Why:**
- SQL MI needs its instance identity to have Directory Readers (or the equivalent Graph permissions) so it can resolve Entra users and service principals. Granting that comes first.
- user1 then needs an Entra server principal (login) in master.
- Creating logins needs ALTER ANY LOGIN, which comes with the **securityadmin** role. Without it, user1 can only connect.
- "Create a managed identity" isn't needed: Microsoft Learn states "An SMI is automatically assigned to Azure SQL Managed Instance when it's created."
- `CREATE USER ... FROM LOGIN` only adds a database user, which gives no server-level permission to create logins.

**Source-answer check:** NOTE hand-numbered "Create managed identity → Grant read access → CREATE LOGIN" ✘. That sequence leaves user1 unable to create logins (no securityadmin / ALTER ANY LOGIN), and the MI identity already exists by default.
**Confidence:** Medium (no community consensus could be found; the answer is based on documented permission requirements)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity ; https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure

---

### Q90. Synapse Pool1: MFA and database-level auth
<sub>ID U-096</sub>

**Sources:** MD-094, CQ-074, NOTE-17   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription that is linked to a hybrid Microsoft Entra (Azure AD) tenant. The subscription contains an Azure Synapse Analytics SQL pool named Pool1.
You need to recommend an authentication solution for Pool1. The solution must support multi-factor authentication (MFA) and database-level authentication.
Which authentication solution or solutions should you include in the recommendation? To answer, select the appropriate options in the answer area.

**Options:**
MFA: Microsoft Entra (Azure AD) authentication | Microsoft SQL Server authentication | Passwordless authentication | Windows authentication
Database-level authentication: Application roles | Contained database users | Database roles | Microsoft SQL Server logins

**✅ Answer:** MFA: Microsoft Entra authentication; Database-level authentication: Contained database users

**Why:** Only Microsoft Entra authentication supports MFA (Universal/Interactive with MFA, enforced by Conditional Access). SQL authentication is password-only, and Windows authentication isn't available in Azure SQL or Synapse. Microsoft Entra authentication "uses contained database users to authenticate identities at the database level". Logins are server-level, and database or application roles are for authorization, not authentication.
*Note:* the product here is Synapse, but the same behavior applies to Azure SQL Database and Managed Instance.

**Source-answer check:** All sources agree ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-overview ; https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure

---

### Q91. ADatum DB3: audit all changes — first step
<sub>ID U-097</sub>

**Sources:** MD-146   **Type:** MCQ   **Case study:** ADatum v1 (financial services)

**Question:**
(Case study: ADatum Corporation, financial services. DB3 on SVR3 (SQL Server 2019) will be migrated to Azure SQL Database. Security requirement: "Ensure that all changes to DB3, including ones within individual transactions, are audited and recorded." Business requirements: minimize costs and administrative effort.)
You need to recommend a solution to meet the security requirements and the business requirements for DB3. What should you recommend as the first step of the solution?

**Options:**
A. Run the sp_addarticle stored procedure.
B. Run the ALTER TABLE statement and specify the ENABLE CHANGE_TRACKING clause.
C. Run the ALTER DATABASE statement and specify the SET CHANGE_TRACKING = ON clause.
D. Run the sys.sp_cdc_enable_db stored procedure.

**✅ Answer:** D

**Why:** Change data capture reads the transaction log and records every insert, update and delete, with before/after values, including multiple changes to the same row inside one transaction. Change tracking (B, C) records only that a row changed (net change, primary key only), so it can't record every change within a transaction. CDC must be enabled at the database level with `sys.sp_cdc_enable_db` before `sys.sp_cdc_enable_table`, so that's the first step. sp_addarticle (A) is for transactional replication and isn't a change-auditing feature.

**Source-answer check:** MD said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/change-data-capture-overview

---

### Q92. SIM: enable CDC for db1
<sub>ID U-098</sub>

**Sources:** MD-209   **Type:** Simulation   **Case study:** —

**Question:**
SIMULATION. You need to enable change data capture (CDC) for db1.
To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:**
(lab task: no options)

**✅ Answer (steps):**
1. In SSMS, connect to the logical server with an account that is **db_owner** of db1 (the server admin works, or the Entra admin if the database was created by an Entra user).
2. Open a new query window against **db1** (not master) and run:
```sql
EXEC sys.sp_cdc_enable_db;
GO
SELECT name, is_cdc_enabled FROM sys.databases WHERE name = 'db1';  -- verify = 1
```
3. Optional, only if the task asks for table-level capture: `EXEC sys.sp_cdc_enable_table @source_schema = N'<schema>', @source_name = N'<table>', @role_name = NULL;`
Prerequisite: db1 must be vCore (any tier) or DTU **S3 or higher**. If you get an error on Basic or S0–S2, scale up first (Portal > db1 > Compute + storage).

**Why:** CDC is enabled per database with `sys.sp_cdc_enable_db`, which needs db_owner. In Azure SQL Database the CDC scheduler replaces SQL Agent jobs, so nothing else has to be configured.

**Source-answer check:** MD said `EXEC sys.sp_cdc_enable_db` in db1 ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/change-data-capture-overview

---

### Q93. Change tracking for webapp1 export
<sub>ID U-099</sub>

**Sources:** MD-108   **Type:** DragDrop   **Case study:** —

**Question:**
You have an Azure subscription that contains an Azure SQL database named SQLDb1. SQLDb1 contains a table named Table1.
You plan to deploy an Azure web app named webapp1 that will export the rows in Table1 that have changed.
You need to ensure that webapp1 can identify the changes to Table1. The solution must meet the following requirements:
- Minimize compute times.
- Minimize storage.

Which three actions should you perform in sequence? To answer, move the appropriate actions from the list of actions to the answer area and arrange them in the correct order.

**Options:**
Actions:
- From webapp1, connect to SQLDb1, obtain the initial dataset and run the CHANGE_TRACKING_CURRENT_VERSION() function.
- Connect to SQLDb1 and run the following Transact-SQL statement: `EXEC sys.sp_cdc_enable_table`
- Connect to SQLDb1 and run the following Transact-SQL statement: `EXEC sys.sp_cdc_enable_db`
- From webapp1, connect to SQLDb1, obtain the initial dataset, and run the CHANGETABLE() function.
- Connect to SQLDb1 and run the following Transact-SQL statement: `ALTER DATABASE SQLDb1 SET CHANGE_TRACKING = ON`
- Connect to SQLDb1 and run the following Transact-SQL statement: `ALTER TABLE dbo.Table1 ENABLE CHANGE_TRACKING`

**✅ Answer:** 1. `ALTER DATABASE SQLDb1 SET CHANGE_TRACKING = ON` → 2. `ALTER TABLE dbo.Table1 ENABLE CHANGE_TRACKING` → 3. From webapp1, obtain the initial dataset and run **CHANGE_TRACKING_CURRENT_VERSION()**.

**Why:**
- Change tracking, not CDC, meets "minimize compute and storage": it stores only the primary keys of changed rows and does no asynchronous log scanning. It must be enabled on the database first, then on the table.
- Microsoft Learn ("Obtain initial data"): "Before an application can obtain changes for the first time, the application must send a query to obtain the initial data and the synchronization version. The application must obtain the appropriate data directly from the table, and then use CHANGE_TRACKING_CURRENT_VERSION() to obtain the initial version." That matches the first action word for word.
- CHANGETABLE(CHANGES …) is used *later*, on each export, with that saved version. It isn't used to get the initial dataset.

**Source-answer check:** MD said step 3 = "obtain the initial dataset and run CHANGETABLE()" ✘. Microsoft Learn ties the initial dataset to CHANGE_TRACKING_CURRENT_VERSION(). Steps 1 and 2 agree ✔. If your answer key follows the dump, note that it uses CHANGETABLE().
**Confidence:** Medium (steps 1 and 2 are certain. Step 3 is disputed: the dump key says CHANGETABLE, while the docs' initial-sync procedure says CHANGE_TRACKING_CURRENT_VERSION.)   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/work-with-change-tracking-sql-server

---

### Q94. SIM: classify StandardCost/ListPrice as Financial/Confidential
<sub>ID U-101</sub>

**Sources:** MD-129   **Type:** Simulation   **Case study:** —

**Question:**
SIMULATION. You need to classify the StandardCost and ListPrice columns of the SalesLT.Product table in db1. The columns must have the following configurations:
- An information type of Financial.
- A sensitivity label of Confidential.

To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:**
(lab task: no options)

**✅ Answer (steps):** Use any one of the following methods.
- **Azure portal:** db1 > **Security > Data Discovery & Classification** > **Classification** tab > **+ Add classification** > Schema **SalesLT**, Table **Product**, Column **StandardCost**, Information type **Financial**, Sensitivity label **Confidential** > **Add classification**. Repeat for **ListPrice**, then click **Save**.
- **SSMS:** right-click db1 > **Tasks > Data Discovery and Classification > Classify Data…** > **Add Classification** (or edit the recommendations for these columns) with the same values for both columns > **Save**.
- **T-SQL** (fastest, run in db1):
```sql
ADD SENSITIVITY CLASSIFICATION TO SalesLT.Product.StandardCost, SalesLT.Product.ListPrice
WITH (LABEL = 'Confidential', INFORMATION_TYPE = 'Financial');
-- verify
SELECT * FROM sys.sensitivity_classifications;
```

**Why:** Data Discovery & Classification stores the label and information type as column metadata. One `ADD SENSITIVITY CLASSIFICATION` statement can classify several columns (documented). With the portal or SSMS, nothing is saved until you click **Save**.

**Source-answer check:** MD (SSMS Classify Data > set both columns > Save) ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/t-sql/statements/add-sensitivity-classification-transact-sql

---

### Q95. Synapse Customers: hide credit card data
<sub>ID U-102</sub>

**Sources:** MD-093, CQ-073   **Type:** MCQ   **Case study:** —

**Question:**
You are designing an enterprise data warehouse in Azure Synapse Analytics that will contain a table named Customers. Customers will contain credit card information.
You need to recommend a solution to provide salespeople with the ability to view all the entries in Customers. The solution must prevent all the salespeople from viewing or inferring the credit card information.
What should you include in the recommendation?

**Options:**
A. row-level security
B. data masking
C. Always Encrypted
D. column-level security

**✅ Answer:** D

**Why:** Column-level security (`GRANT SELECT ON dbo.Customers (col1, col2, …)` without the card column, or DENY on that column) means salespeople can't read or reference the credit card column at all, so they can't infer it with predicates. Dynamic data masking (B) only masks output, and Microsoft Learn warns that users with ad hoc query access can infer masked values by brute force (for example, `WHERE` range probing). Row-level security (A) filters rows, not columns, and would conflict with "view all the entries". Always Encrypted (C) is a heavier solution aimed at hiding data from DBAs and the server, and isn't the minimal fit for this requirement.
*Note:* the product is Synapse, but the same behavior applies to Azure SQL Database and Managed Instance (column-level GRANT/DENY and DDM inference work the same way).

**Source-answer check:** MD & CQ said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-gb/azure/synapse-analytics/sql-data-warehouse/column-level-security ; https://learn.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking

---

### Q96. DDM custom string for Customer_Phone
<sub>ID U-103</sub>

**Sources:** MD-086, CQ-066, NOTE-18   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure SQL database that contains a table named Customer. Customer has the columns shown in the following table.

| Customer_ID | Customer_Name | Customer_Phone |
|---|---|---|
| 11001 | Contoso, Ltd. | 555-555-0173 |
| 11002 | Litware, Inc. | 555-505-3124 |
| 11003 | ADatum Corporation | 555-689-4312 |

You plan to implement a dynamic data mask for the Customer_Phone column. The mask must meet the following requirements:
- The first six numerals of each customer's phone number must be masked.
- The last four digits of each customer's phone number must be visible.
- Hyphens must be preserved and displayed.

How should you configure the dynamic data mask? To answer, select the appropriate options in the answer area.

**Options:**
Exposed Prefix: 0 | 1 | 3 | 5
Padding String: x | xxxxxx | xxx-xxx | xxx-xxx- | x[3]-x[3]
Exposed Suffix: 0 | 1 | 3 | 5

**✅ Answer:** Exposed Prefix: 0; Padding String: xxx-xxx; Exposed Suffix: 5

**Why:** The custom string mask is `partial(prefix,"padding",suffix)` and shows prefix characters + padding + suffix characters. With prefix 0, `555-555` is replaced by the padding `xxx-xxx`. Suffix 5 exposes the last five characters, `-0173`, which includes the hyphen. Result: `xxx-xxx-0173`, which meets all three rules. `xxx-xxx-` would need suffix 4, which isn't offered, and suffix 5 with that padding would produce a double hyphen.

**Source-answer check:** All sources agree ✔ (NOTE circle is on "XXX-XXX", not "XXX-XXX-")
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/dynamic-data-masking-overview ; https://learn.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking

---

### Q97. SIM: mask email in SalesLT.Customer
<sub>ID U-104</sub>

**Sources:** MD-143   **Type:** Simulation   **Case study:** —

**Question:**
SIMULATION. You need to ensure that when non-administrative users query the SalesLT.Customer table in db1, email addresses are obscured. For example, an email address of alice@contoso.com must appear as aXXX@XXXX.com.
To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:**
(lab task: no options)

**✅ Answer (steps):** Use either method.
- **Azure portal:** SQL databases > **db1** > **Security > Dynamic Data Masking** > **+ Add mask** > Schema **SalesLT**, Table **Customer**, Column **EmailAddress** > Masking field format **Email (aXXX@XXXX.com)** > **Add** > **Save**.
- **T-SQL in db1:**
```sql
ALTER TABLE SalesLT.Customer
ALTER COLUMN EmailAddress ADD MASKED WITH (FUNCTION = 'email()');
```
Admins (server admin, Entra admin, db_owner) still see the real values. Non-admins see `aXXX@XXXX.com`.

**Why:** The DDM email() function shows the first letter and replaces the rest with the constant pattern `XXX@XXXX.com`, which exactly matches the required output. (The AdventureWorksLT column is named EmailAddress.)

**Source-answer check:** MD (portal DDM, Email format, Save) ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/dynamic-data-masking-overview

---

### Q98. Synapse Email column mask from portal
<sub>ID U-105</sub>

**Sources:** MD-006, CQ-023, NOTE-043   **Type:** MCQ   **Case study:** —

**Question:**
You have a SQL pool in Azure Synapse that contains a table named dbo.Customers. The table contains a column named Email.
You need to prevent nonadministrative users from seeing the full email addresses in the Email column. The users must see values in a format of aXXX@XXXX.com instead.
What should you do?

**Options:**
A. From the Azure portal, set a mask on the Email column.
B. From the Azure portal, set a sensitivity classification of Confidential for the Email column.
C. From Microsoft SQL Server Management Studio, set an email mask on the Email column.
D. From Microsoft SQL Server Management Studio, grant the SELECT permission to the users for all the columns in the dbo.Customers table except Email.

**✅ Answer:** A

**Why:** Dynamic data masking with the Email format gives exactly `aXXX@XXXX.com`, and it is configured on the dedicated SQL pool's **Dynamic Data Masking** blade in the Azure portal. Classification (B) only labels the column and doesn't hide data. SSMS has no masking designer (C). A mask can be added from SSMS only by typing T-SQL `ALTER COLUMN … ADD MASKED`, which isn't what the option describes, so the exam key is A. Column-level GRANT (D) hides the column entirely instead of showing the masked format.
*Note:* the product is Synapse, but the same behavior applies to Azure SQL Database (portal Dynamic Data Masking blade, Email mask).

**Source-answer check:** All sources say A ✔
**Confidence:** Medium (A is the established key. C is technically possible via T-SQL in SSMS, which is the only ambiguity.)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/dynamic-data-masking-overview ; https://learn.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking

---

### Q99. Cell-level encryption of Col1 sequence
<sub>ID U-108</sub>

**Sources:** MD-102   **Type:** DragDrop   **Case study:** —

**Question:**
You have an Azure SQL database named DB1. DB1 contains a table that has a column named Col1. You need to encrypt the data in Col1. Which four actions should you perform for DB1 in sequence? (Move the appropriate actions to the answer area and arrange them in the correct order.)

**Options:**
Actions:
- Create a database master key.
- Create a column master key.
- Open the symmetric key.
- Create a certificate.
- Update Col1.
- Create a symmetric key.

**✅ Answer:** 1. Create a database master key → 2. Create a certificate → 3. Create a symmetric key → 4. Open the symmetric key

**Why:** Cell-level (column-level) symmetric encryption needs a DMK (to protect the certificate's private key), a certificate, a symmetric key encrypted by that certificate, and then `OPEN SYMMETRIC KEY ... DECRYPTION BY CERTIFICATE` before `EncryptByKey()` can be used in an UPDATE. "Create a column master key" belongs to Always Encrypted (a different feature, keys held outside the database), so it is a distractor. "Update Col1" is the step that writes encrypted data after the key is open; with only four slots and no DMK stated as already existing, the accepted sequence starts with the DMK and ends with opening the key.

**Source-answer check:** MD-102 said DMK → certificate → symmetric key → open symmetric key ✔. (Some candidates argue for certificate → symmetric key → open → update Col1 on the theory that a DMK already exists; the docs say to create the DMK first if the database doesn't have one, so the DMK-first key is the stronger answer.)
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/encrypt-a-column-of-data

---

### Q100. Firewall series: database-level rules in each DB
<sub>ID U-109</sub>

**Sources:** MD-138   **Type:** Yes/No series   **Case study:** —

**Question:**
You have the on-premises networks shown in the following table.

| Name | Location | Subnet |
|---|---|---|
| Branch1 | Sydney | 131.107.10.0/24 |
| Branch2 | Melbourne | 131.107.11.0/24 |

You have an Azure subscription that contains an Azure SQL Database server named SQL1. SQL1 contains two databases named DB1 and DB2. You need to configure access to DB1 and DB2. The solution must meet the following requirements:
- Ensure that DB1 can be accessed only by users in Branch1.
- Ensure that DB2 can be accessed only by users in Branch2.

Solution: You connect to DB1 and run:
```sql
EXECUTE sp_set_database_firewall_rule 'Allow db1 users', '131.107.10.0', '131.107.10.255';
```
You connect to DB2 and run:
```sql
EXECUTE sp_set_database_firewall_rule 'Allow db2 users', '131.107.11.0', '131.107.11.255';
```
Does this meet the goal?

**Options:**
A. Yes
B. No

**✅ Answer:** A (Yes)

**Why:** Database-level IP firewall rules are created with `sp_set_database_firewall_rule` while connected to the specific user database, and they grant access only to that database. Running it in DB1 for 131.107.10.0/24 and in DB2 for 131.107.11.0/24 isolates each database to its branch without a server-level rule that would open both databases.

**Source-answer check:** MD-138 said Yes ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure

---

### Q101. Firewall series: sp_set_firewall_rule run in DB1
<sub>ID U-110</sub>

**Sources:** MD-134   **Type:** Yes/No series   **Case study:** —

**Question:**
(Same scenario as Q100: Branch1 Sydney 131.107.10.0/24; Branch2 Melbourne 131.107.11.0/24; SQL1 hosts DB1 and DB2; DB1 must be accessible only by Branch1 users, DB2 only by Branch2 users.)

Solution: You connect to DB1 and run:
```sql
EXECUTE sp_set_firewall_rule 'Allow db1 users', '131.107.10.0', '131.107.10.255';
```
You connect to DB2 and run:
```sql
EXECUTE sp_set_database_firewall_rule 'Allow db2 users', '131.107.11.0', '131.107.11.255';
```
Does this meet the goal?

**Options:**
A. Yes
B. No

**✅ Answer:** B (No)

**Why:** `sp_set_firewall_rule` creates server-level rules and can only be executed in the `master` database. Run in DB1 it fails, and even in master it would open every database on SQL1 (including DB2) to Branch1. Only the DB2 half of the solution is correct.

**Source-answer check:** MD-134 said No ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure

---

### Q102. Firewall series: "Deny" rules on DB1/DB2
<sub>ID U-111</sub>

**Sources:** MD-136   **Type:** Yes/No series   **Case study:** —

**Question:**
(Same scenario as Q100.)

Solution: You connect to DB1 and run:
```sql
EXECUTE sp_set_firewall_rule 'Deny db1 users', '131.107.11.0', '131.107.11.255';
```
You connect to DB2 and run:
```sql
EXECUTE sp_set_database_firewall_rule 'Deny db2 users', '131.107.10.0', '131.107.10.255';
```
Does this meet the goal?

**Options:**
A. Yes
B. No

**✅ Answer:** B (No)

**Why:** Azure SQL IP firewall rules are allow-only; naming a rule "Deny…" doesn't make it a deny rule. The DB2 rule would actually allow Branch1 into DB2, the reverse of the requirement. Also, `sp_set_firewall_rule` is a server-level procedure that runs only in `master`, so the DB1 command fails.

**Source-answer check:** MD-136 said No ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure

---

### Q103. Firewall series: master server rule 131.107.10.0/24
<sub>ID U-112</sub>

**Sources:** MD-137   **Type:** Yes/No series   **Case study:** —

**Question:**
(Same scenario as Q100.)

Solution: You connect to the master database of SQL1 and run:
```sql
EXECUTE sp_set_firewall_rule 'Allow db1 and db2 users', '131.107.10.0', '131.107.10.255';
```
Does this meet the goal?

**Options:**
A. Yes
B. No

**✅ Answer:** B (No)

**Why:** A server-level rule grants the range access to all databases on SQL1. Branch1 could reach DB2, and Branch2 gets no access at all. Use `sp_set_database_firewall_rule` in each database instead.

**Source-answer check:** MD-137 said No ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure

---

### Q104. Firewall series: master server rule 131.107.11.0/24
<sub>ID U-113</sub>

**Sources:** MD-135   **Type:** Yes/No series   **Case study:** —

**Question:**
(Same scenario as Q100.)

Solution: You connect to the master database of SQL1 and run:
```sql
EXECUTE sp_set_firewall_rule 'Allow db1 and db2 users', '131.107.11.0', '131.107.11.255';
```
Does this meet the goal?

**Options:**
A. Yes
B. No

**✅ Answer:** B (No)

**Why:** The server-level rule lets Branch2 reach both DB1 and DB2 and gives Branch1 no access, so neither requirement is met. Per-database rules (`sp_set_database_firewall_rule`) are required.

**Source-answer check:** MD-135 said No ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure

---

### Q105. Azure CLI rule allowing Azure services
<sub>ID U-114</sub>

**Sources:** MD-133   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription that contains a resource group named RG1. RG1 contains an Azure SQL Server named Server1 in the West US Azure region. You need to ensure that any Azure services deployed to RG1 can access Server1. How should you complete the Azure CLI command?
```
az sql server [Box1] create \
  --resource-group rg1 \
  --server server1 \
  --name AllowAzureServices \
  [Box2]
```

**Options:**
Box 1: firewall-rule | ipv6-firewall-rule | outbound-firewall-rule | vnet-rule
Box 2: --outbound-rule-fqdn allowedFQDN | --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0 | --start-ip-address 13.86.217.0 --end-ip-address 13.86.217.255 | --start-ipv6-address 9a41:a145:2a80:6c8d:4628:a1b3:5812:3283 --end-ipv6-address 9a41:a145:2a80:6c8d:4628:a1b3:5812:3283

**✅ Answer:** Box 1: firewall-rule; Box 2: --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

**Why:** An IPv4 server firewall rule with start and end address 0.0.0.0 is the special rule (shown in the portal as "Allow Azure services and resources to access this server", AllowAllWindowsAzureIps) that lets Azure-hosted services connect. vnet-rule needs a subnet/service endpoint, and outbound-firewall-rule controls egress, not inbound. Note: the docs warn this rule admits all Azure services, including other customers' subscriptions, not just RG1. It is still the only option here that meets the requirement.

**Source-answer check:** MD-133 said firewall-rule + 0.0.0.0/0.0.0.0 ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure

---

### Q106. DB1 access limited to 10 VMs in Subnet1
<sub>ID U-115</sub>

**Sources:** MD-131   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure SQL database named DB1. You have 10 Azure virtual machines that connect to a virtual network subnet named Subnet1. You need to implement a database-level firewall that meets the following requirements:
- Ensures that only the 10 virtual machines can access DB1.
- Follows the principle of least privilege.

How should you configure the firewall rule, and how should you establish network connectivity from the virtual machines to DB1?

**Options:**
Firewall rule: Add your client IPv4 address. | Allow Azure services and resources to access DB1. | Allow traffic from a specific virtual network.
Network connectivity: Assign static public IP addresses to the virtual machines. | Create a private endpoint. | Create a service endpoint.

**✅ Answer:** Firewall rule: **Allow traffic from a specific virtual network**; Network connectivity: **Create a service endpoint**

**Why:** Subnet1 contains exactly the 10 VMs, so a virtual network rule for Subnet1 admits only those VMs (least privilege) without tracking individual IPs. A VNet rule requires the **Microsoft.Sql service endpoint** on the subnet: "to create a virtual network rule, there must first be a virtual network service endpoint". The other options fail. "Allow Azure services" admits every Azure tenant's resources. A client-IPv4 rule doesn't match VM traffic once it uses a service endpoint, and static public IPs plus IP rules add management overhead. A private endpoint doesn't use firewall rules at all.
Caveat: VNet rules are technically **server-level**, while the stem says "database-level firewall". The stem is loose, but this is the only consistent pairing among the options.

**Source-answer check:** MD-131 said "Add your client IPv4 address" ✘ + "Create a service endpoint" ✔. An IPv4 rule and a service endpoint don't work together; MD's own explanation quotes the VNet-rule doc.
**Confidence:** Low (poorly worded item. The alternative reading, per-VM IP rules plus static public IPs, satisfies "database-level" literally but is clumsy and not the coherent design.)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/vnet-service-endpoint-rule-overview ; https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure

---

### Q107. Ledger + SYSTEM_VERSIONING table statements
<sub>ID U-118</sub>

**Sources:** MD-116   **Type:** Statements (Yes/No)   **Case study:** —

**Question:**
You have an Azure SQL logical server. You run the following script.
```sql
CREATE DATABASE Sales
GO
CREATE TABLE [dbo].[Orders]
  (
    [OrderID] INT NOT NULL,
    [OrderDescription] NVARCHAR (MAX) NOT NULL,
    [Timestamp] Datetime2 NOT NULL
  )
  WITH (
    SYSTEM_VERSIONING = ON,
    LEDGER = ON
  );
GO
```
For each of the following statements, select Yes if the statement is true. Otherwise, select No.

**Options:**
Statements:
1. The Orders table will allow only rows to be inserted.
2. To create additional tables in the Sales database, the LEDGER = ON parameter must be used.
3. To ensure that a timestamp is added to each row in the Orders table, the GENERATED ALWAYS column must be added to the script.

**✅ Answer:** 1: No; 2: No; 3: Yes

**Why:** (1) Without `APPEND_ONLY = ON`, `SYSTEM_VERSIONING = ON, LEDGER = ON` creates an *updatable* ledger table, so updates and deletes are allowed and old versions go to the history table. (2) The database wasn't created `WITH LEDGER = ON`, so it isn't a ledger database. Other tables can be ordinary tables, and LEDGER = ON is only needed if you want them to be ledger tables. (3) The columns ledger adds automatically are transaction IDs and sequence numbers (bigint), not timestamps. The `[Timestamp]` column is an ordinary column the app must fill. To have the system stamp each row, you add a `datetime2 GENERATED ALWAYS AS ROW START/END` period (the table can be both ledger and temporal).

**Source-answer check:** MD-116 said No / No / Yes ✔
**Confidence:** Medium (statement 3 is worded loosely; statements 1 and 2 are doc-confirmed)   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/ledger/ledger-updatable-ledger-tables ; https://learn.microsoft.com/en-us/sql/relational-databases/security/ledger/ledger-limits

---

### Q108. ADatum remote developers access to Instance1
<sub>ID U-119</sub>

**Sources:** MD-077   **Type:** MCQ   **Case study:** ADatum v1 (financial services)

**Question:**
(Case: ADatum plans to deploy SQL managed instance Instance1 to Network1 (East US) and migrate DB1/DB2 to it. Afterwards, database development is handed to remote developers who use Microsoft Entra joined Windows 11 devices. Security requirement: "Ensure that only designated developers who use Microsoft Entra joined Windows 11 devices can access DB1 and DB2 remotely." Business: minimize cost and administrative effort.)

You need to recommend a solution that will enable remote developers to access DB1 and DB2. The solution must support the planned changes and meet the security requirements. What should you include in the recommendation?

**Options:**
A. a public endpoint via a database-level firewall rule
B. a Point-to-Site (P2S) VPN
C. a public endpoint via a server-level firewall rule
D. a private endpoint

**✅ Answer:** B

**Why:** The developers are remote individual clients outside Azure. A P2S VPN to Network1 lets each device reach Instance1's VNet-local endpoint. With Microsoft Entra ID authentication (Azure VPN Client on Windows) and Conditional Access, it can be limited to designated users on Entra-joined/compliant devices. A private endpoint only projects the MI into another Azure VNet, so remote laptops still need a VPN to reach it. A and C don't apply to SQL Managed Instance, which has no server- or database-level IP firewall rules (its public endpoint is controlled by NSG), and an IP rule can't identify the user or device.

**Source-answer check:** MD-077 said D ✘. A private endpoint on its own gives remote devices no path to the VNet and can't enforce the Entra-joined-device requirement.
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/azure/vpn-gateway/point-to-site-entra-gateway ; https://learn.microsoft.com/en-us/azure/vpn-gateway/point-to-site-about

---

### Q109. Power BI to MI: public endpoint + NSG
<sub>ID U-120</sub>

**Sources:** MD-120   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription that contains an Azure SQL managed instance named SQL1. You use the Microsoft Power BI service. You need to configure connectivity from the Power BI service to SQL1. The solution must ensure that only the Power BI service can initiate connections to SQL1. Which type of endpoint should you use for SQL1, and what should you use to control the connections to SQL1?

**Options:**
Endpoint type: Private | Public | Service
Control connections by using: A database-level firewall | A network security group (NSG) | A server-level firewall | Virtual network firewall rules

**✅ Answer:** Endpoint type: Public; Control connections by using: A network security group (NSG)

**Why:** The Power BI service is a multi-tenant cloud service outside your VNet, so (without a gateway) it reaches a managed instance through the MI public endpoint (port 3342). On MI, access to that endpoint is controlled by inbound NSG rules on the MI subnet. Use an Allow rule for TCP 3342 with source service tag `PowerBI`, so only Power BI can initiate connections. MI has no server/database-level IP firewall or VNet firewall rules (those are Azure SQL Database features), and MI doesn't use "service" endpoints.

**Source-answer check:** MD-120 said Public + NSG ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/public-endpoint-configure?view=azuresql ; https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/public-endpoint-overview?view=azuresql

---

### Q110. Private IP and single-VNet access: Private Link
<sub>ID U-121</sub>

**Sources:** MD-012, CQ-029   **Type:** MCQ   **Case study:** —

**Question:**
You plan to deploy an app that includes an Azure SQL database and an Azure web app. The app has the following requirements:
- The web app must be hosted on an Azure virtual network.
- The Azure SQL database must be assigned a private IP address.
- The Azure SQL database must allow connections only from a specific virtual network.

You need to recommend a solution that meets the requirements. What should you include in the recommendation?

**Options:**
A. Azure Private Link
B. a network security group (NSG)
C. a database-level firewall
D. a server-level firewall

**✅ Answer:** A

**Why:** Azure Private Link creates a private endpoint, a NIC with a private IP from a subnet of the chosen VNet, for the logical server. With public network access disabled, the database is reachable only through that VNet (and networks connected to it). NSGs filter subnet traffic but can't give the database a private IP. Server- and database-level IP firewall rules act on the public endpoint and don't assign a private IP.

**Source-answer check:** MD-012 & CQ-029 said A ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/private-endpoint-overview?view=azuresql

---

### Q111. SIM: connect to db1 via private IP on VNET1
<sub>ID U-122</sub>

**Sources:** MD-142   **Type:** Simulation (lab)   **Case study:** —

**Question:**
SIMULATION. You need to ensure that you can connect to db1 by using a private IP address on a virtual network named VNET1. You may need to use SQL Server Management Studio and the Azure portal.

**Options:**
— (lab task)

**✅ Answer:** Create a private endpoint for the logical server that hosts db1, in a subnet of VNET1.
1. Azure portal → open the **SQL server** (logical server) that hosts db1 → **Security › Networking** → **Private access** tab → **+ Create a private endpoint**. (Equivalent: search **Private endpoints** → **+ Create**.)
2. **Basics:** subscription and resource group (same as the server's), a name such as `db1-pe`, and the region of VNET1.
3. **Resource:** Resource type **Microsoft.Sql/servers**, Resource = the server hosting db1, Target sub-resource **sqlServer**.
4. **Virtual Network:** Virtual network **VNET1**, choose a subnet (dynamic IP allocation is fine).
5. **DNS:** Integrate with private DNS zone = **Yes** (`privatelink.database.windows.net`) so `<server>.database.windows.net` resolves to the private IP inside VNET1.
6. **Review + create** → **Create**. Wait until the endpoint's connection state is **Approved** (auto-approved because you own the server).
7. (Optional hardening) On the server's Networking page, set **Public network access** to **Disable** so only private connections work.
8. Verify: from a VM in VNET1, `nslookup <server>.database.windows.net` returns a private IP; connect with SSMS to `<server>.database.windows.net` / db1.

**Why:** A private endpoint (Azure Private Link) assigns the logical server a private IP from VNET1. A service endpoint or VNet rule would still use the server's public IP.

**Source-answer check:** MD-142 said create private endpoint, resource type Microsoft.Sql/servers, VNET1 ✔ (added the target sub-resource, DNS-zone and approval details).
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/private-endpoint-overview?view=azuresql

---

### Q112. VM1 may reach only SqlSrv1: what to create on VNet1
<sub>ID U-123</sub>

**Sources:** MD-100, CQ-080, MD-091, CQ-071   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure virtual machine named VM1 on a virtual network named VNet1. Outbound traffic from VM1 to the internet is blocked. You have an Azure SQL database named SqlDb1 on a logical server named SqlSrv1. You need to implement connectivity between VM1 and SqlDb1 to meet the following requirements:
- Ensure that VM1 cannot connect to any Azure SQL Server other than SqlSrv1.
- Restrict network connectivity to SqlSrv1.

What should you create on VNet1?

**Options:**
A. a VPN gateway
B. a service endpoint
C. a private endpoint
D. an ExpressRoute gateway

**✅ Answer:** C

**Why:** A private endpoint maps one specific resource (SqlSrv1) into VNet1. With internet outbound blocked, VM1 can reach only that server, which protects against data exfiltration to other servers. A Microsoft.Sql service endpoint opens a route to all Azure SQL servers in the region, not just SqlSrv1. VPN and ExpressRoute gateways are for on-premises/cross-network connectivity.

**Variant seen in other dumps (MD-091, CQ-071):** same question with options `A. a VPN gateway / B. a service endpoint / C. a private link / D. an ExpressRoute gateway` → answer **C**.

**Source-answer check:** MD-100 & CQ-080 said C ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/private-endpoint-overview?view=azuresql

---

### Q113. Block public endpoint, minimize exfiltration: Private Link
<sub>ID U-125</sub>

**Sources:** MD-084, CQ-064   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure virtual machine named VM1 on a virtual network named VNet1. Outbound traffic from VM1 to the internet is blocked. You have an Azure SQL database named SqlDb1 on a logical server named SqlSrv1. You need to implement connectivity between VM1 and SqlDb1 to meet the following requirements:
- Ensure that all traffic to the public endpoint of SqlSrv1 is blocked.
- Minimize the possibility of VM1 exfiltrating data stored in SqlDb1.

What should you create on VNet1?

**Options:**
A. a VPN gateway
B. a service endpoint
C. a private link
D. an ExpressRoute gateway

**✅ Answer:** C

**Why:** With Private Link, VM1 reaches SqlSrv1 through a private endpoint, so the server's public network access can be set to **Deny/Disabled**, blocking all public-endpoint traffic. Because the endpoint maps only SqlSrv1, VM1 can't push data to other SQL servers (exfiltration protection). A service endpoint still uses the server's public endpoint and allows every SQL server in the region.

**Source-answer check:** MD-084 & CQ-064 said C ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/private-endpoint-overview?view=azuresql

---

### Q114. VM1 (West US 2) to MI1 (East US) low latency
<sub>ID U-126</sub>

**Sources:** NOTE-187   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription that contains the resources shown in the following table.

| Name | Type | Azure region |
|---|---|---|
| VM1 | Azure virtual machine | West US 2 |
| MI1 | Azure SQL Managed Instance | East US |

You need to configure a connection between VM1 and MI1. The solution must meet the following requirements:
- The connection must be encrypted.
- Network latency must be minimized.

What should you implement?

**Options:**
A. virtual network peering
B. private endpoints
C. service endpoints
D. a site-to-site VPN

**✅ Answer:** A

**Why:** Global virtual network peering connects the two VNets over the Microsoft backbone, so the docs say there is "no noticeable latency penalty". Clients can use MI's VNet-local endpoint with the low-latency redirect connection type, and the TDS session itself is TLS-encrypted. Private endpoints to MI always use the proxy connection type (extra hop), so latency is higher. A site-to-site VPN adds gateway/IPsec overhead and is meant for on-premises links. Service endpoints don't apply to SQL Managed Instance.

**Source-answer check:** NOTE-187 said A ✔
**Confidence:** Medium (the docs call private endpoints the *most secure* option, but the question optimizes for latency, and peering gives the direct, redirect-capable path)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/connect-application-instance ; https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/private-endpoint-overview?view=azuresql

---

### Q115. Contoso customers create own objects
<sub>ID U-127</sub>

**Sources:** MD-081, NOTE-127   **Type:** MCQ   **Case study:** Contoso

**Question:**
(Case business goal: "Ensure that users of the PaaS solution can create their own database objects but be prevented from modifying any of the existing database objects supplied by Contoso." Also: use built-in features, minimize development effort.)

You need to recommend a solution to ensure that the customers can create the database objects. The solution must meet the business goals. What should you include in the recommendation?

**Options:**
A. For each customer, grant the customer ddl_admin to the existing schema.
B. For each customer, create an additional schema and grant the customer ddl_admin to the new schema.
C. For each customer, create an additional schema and grant the customer db_writer to the new schema.
D. For each customer, grant the customer db_writer to the existing schema.

**✅ Answer:** B

**Why:** Customers need DDL rights (CREATE/ALTER objects), which db_datawriter doesn't give: it only allows INSERT/UPDATE/DELETE. So C and D are out. Granting DDL rights on the existing (Contoso) schema would let them alter or drop Contoso's objects (A). A separate schema per customer, owned by or with ALTER/CREATE rights for that customer, isolates their objects from Contoso's. (Strictly, db_ddladmin is a database-wide role, so in practice you grant CREATE TABLE etc. plus ALTER on the new schema, but B is the intended option.)

**Source-answer check:** MD-081 & NOTE-127 said B ✔ (NOTE-127's case is unnamed, but the stem and options are identical, so it maps to Contoso)
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles

---

### Q116. Contoso DBAGroup1/DBAGroup2 role assignments
<sub>ID U-128</sub>

**Sources:** MD-080   **Type:** Statements (Yes/No)   **Case study:** Contoso

**Question:**
You are evaluating the role assignments. For each customer's Azure SQL Database server you plan these assignments (scope: This resource):

| Role | Name | Type | Scope |
|---|---|---|---|
| Contributor | DBAGroup1 | Group | This resource |
| SQL DB Contributor | DBAGroup2 | Group | This resource |

For each of the following statements, select Yes if the statement is true. Otherwise, select No.

**Options:**
Statements:
1. DBAGroup1 will be able to sign in to each customer's Azure SQL database by using Azure Data Studio.
2. DBAGroup1 will be able to assign the SQL DB Contributor role to other users.
3. DBAGroup2 will be able to create a new Azure SQL database on each customer's Azure SQL Database server.

**✅ Answer:** 1: No; 2: No; 3: Yes

**Why:** (1) Azure RBAC roles act on the control plane only. Signing in to the database needs a SQL login/contained user or the Entra admin, which Contributor doesn't grant. (2) Contributor excludes `Microsoft.Authorization/*/Write`, so it can't create role assignments (that needs Owner or User Access Administrator). (3) SQL DB Contributor includes `Microsoft.Sql/servers/databases/*` ("Create and manage SQL databases") and is assigned at the server scope, so DBAGroup2 can create databases on that server (but not access their data).

**Source-answer check:** MD-080 said No / No / Yes ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/databases

---

### Q117. Entra User1 add/remove columns, least privilege
<sub>ID U-129</sub>

**Sources:** MD-106   **Type:** MultiSelect (choose two)   **Case study:** —

**Question:**
You have an Azure SQL database named DB1. A user named User1 has a Microsoft Entra account. You need to provide User1 with the ability to add and remove columns from the tables in DB1. The solution must use the principle of least privilege. Which two actions should you perform? Each correct answer presents part of the solution.

**Options:** *(RECONSTRUCTED. The source has no option list or answer. These are the option set this item is commonly published with, but they couldn't be confirmed online.)*
A. Assign the database user the db_owner role.
B. Assign the database user the db_ddladmin role.
C. Create a contained database user.
D. Create a login and an associated database user.

**✅ Answer:** B and C

**Why:** For an Entra identity in Azure SQL Database, you create a contained database user in DB1 (`CREATE USER [user1@contoso.com] FROM EXTERNAL PROVIDER;`). No server login in master is needed, which keeps access scoped to DB1 only (fewer privileges than D). Adding or dropping columns is DDL (`ALTER TABLE`), which db_ddladmin allows (`ALTER ROLE db_ddladmin ADD MEMBER [user1@contoso.com];`). db_owner (CONTROL DATABASE) gives far more than needed, which violates least privilege.

**Source-answer check:** MD-106 has no answer ("See explanation" with nothing following), so there's nothing to compare. The answer is reasoned from docs against the reconstructed options.
**Confidence:** Medium (the answer logic is doc-backed; the option list is reconstructed)   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles

---

### Q118. Test impersonation with EXECUTE AS USER / REVERT
<sub>ID U-130</sub>

**Sources:** MD-101, NOTE-208   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure SQL database named db1 that contains a Microsoft Entra (Azure AD) user named user1. You need to test impersonation of user1 in db1 by running a SELECT statement and returning to the original execution context. How should you complete the Transact-SQL statement?

```sql
EXECUTE AS [Box 1] = 'user1@contoso.com'
GO
SELECT SUSER_SNAME()
[Box 2]
GO
```

**Options:**
Box 1: CALLER / LOGIN / OWNER / USER
Box 2: REVERT / REVOKE / ROLLBACK

**✅ Answer:** Box 1: USER; Box 2: REVERT

**Why:** In Azure SQL Database, EXECUTE AS LOGIN is not available (server-scope impersonation isn't supported), so a database user is impersonated with EXECUTE AS USER. REVERT switches the context back to the previous caller. CALLER has no effect outside a module; OWNER is only valid in a module's WITH EXECUTE AS clause; REVOKE removes permissions and ROLLBACK undoes a transaction — neither resets execution context.

**Source-answer check:** MD and NOTE both said USER / REVERT ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/t-sql/statements/execute-as-transact-sql

---

### Q119. Grant group read on future tables (schema)
<sub>ID U-131</sub>

**Sources:** MD-111   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription that contains a group named Group1 and an Azure SQL managed instance that hosts a database named DB1. You need to ensure that Group1 has read access to new tables created in DB1. The solution must use the principle of least privilege. How should you complete the Transact-SQL statement?

```sql
GRANT [Box 1] ON [Box 2] :: [dbo] TO [Contoso\group1]
```
(The source transcribes the target as `::[table]`; the securable after `::` is the schema name.)

**Options:**
Box 1: DELETE / INSERT / SELECT / UPDATE
Box 2: DATABASE / SCHEMA / TABLE

**✅ Answer:** Box 1: SELECT; Box 2: SCHEMA

**Why:** SELECT is the read permission. A permission granted on a schema applies to every object in that schema, including tables created later, so it meets the "new tables" requirement. A table-level grant would not cover new tables. A database-level grant would also cover them but is broader than necessary, so SCHEMA is the least-privilege scope.

**Source-answer check:** MD said SELECT / Schema ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/t-sql/statements/grant-schema-permissions-transact-sql

---

### Q120. SIM: role that reads one table only
<sub>ID U-132</sub>

**Sources:** MD-128   **Type:** Simulation (lab)   **Case study:** —

**Question:**
SIMULATION. You need to create a new database role named role1 that can read the contents of the SalesLT.SalesOrderDetail table only in db1. To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:** none (lab task)

**✅ Answer:** Create a **database** role in db1 and grant it SELECT on that one table only:
1. In SSMS, connect to the logical server and open a new query window **in db1** (not master).
2. Run:
```sql
CREATE ROLE role1;
GRANT SELECT ON OBJECT::SalesLT.SalesOrderDetail TO role1;
```
3. (GUI alternative) db1 > Security > Roles > Database Roles > New Database Role. Set Role name = role1, go to Securables > Search > Specific objects > Tables > SalesLT.SalesOrderDetail, tick **Grant** for **Select**, then OK.
4. Optional check: `SELECT * FROM sys.database_permissions WHERE grantee_principal_id = DATABASE_PRINCIPAL_ID('role1');`

**Why:** A user-defined database role with an object-level SELECT grant gives read access to that table and nothing else. Do not add the role to db_datareader, because that grants read on all tables. A server role is the wrong kind of object: it is server-scoped, and the task asks for a role inside db1. Azure SQL Database also offers only fixed server roles, not user-defined ones.

**Source-answer check:** MD described the SSMS "New Server Role" dialog ✘. The task asks for a database role, and a server role can't be scoped to a table in db1. The intent (grant Select on that one table) is right.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles

---

### Q121. Allow master user to create databases
<sub>ID U-133</sub>

**Sources:** MD-104   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription that contains a logical SQL server named Server1. The master database of Server1 contains a user named User1. You need to ensure that User1 can create databases on Server1. Which database role should you assign to User1?

**Options:**
A. db_owner
B. dbmanager
C. dbo
D. db_ddladmin

**✅ Answer:** B

**Why:** dbmanager is a special role that exists only in the virtual master database of Azure SQL Database. Its members can create and delete databases, and the creator becomes the dbo of each new database. db_owner and db_ddladmin are per-database roles and don't grant CREATE DATABASE on the logical server. dbo is a user, not a role. Note: the user should be login-based, because the docs say contained users in master added to dbmanager can't create databases.

**Source-answer check:** MD said B ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles

---

### Q122. db_datawriter capabilities and read role
<sub>ID U-134</sub>

**Sources:** MD-103   **Type:** Hotspot   **Case study:** —

**Question:**
You have a Microsoft SQL Server database named DB1 that contains a table named Table1. The database role membership for a user named User1 is shown in the exhibit.

Exhibit (SSMS "Database User - User1" > Membership): **checked: db_datawriter only**. Unchecked: db_accessadmin, db_backupoperator, db_datareader, db_ddladmin, db_denydatareader, db_denydatawriter, db_owner, db_securityadmin, plus custom roles (External Sales, Far West Sales, Great Lakes Sales, Mideast Sales, New England Sales, Plains Sales, Rocky Mountain Sales, SalesSchemaDeny, SalesSchemaRead, Southeast Sales, Southwest Sales, WebsiteExecute).

Use the drop-down menus to select the answer choice that completes each statement.

**Options:**
Statement 1: User1 can [add a column to Table1 | delete a row from Table1 | drop Table1].
Statement 2: To ensure that User1 can run queries to retrieve data from DB1, you must assign User1 the [db_datareader | db_ddladmin | db_denydatareader | db_denydatawriter] database role.

**✅ Answer:** Statement 1: delete a row from Table1; Statement 2: db_datareader

**Why:** db_datawriter grants INSERT, UPDATE and DELETE on all user tables, so User1 can delete rows. Adding a column or dropping the table is DDL, which needs db_ddladmin, ALTER or CONTROL. db_datareader grants SELECT on all user tables and views. db_denydatareader would block reads, and db_ddladmin doesn't grant SELECT.

**Source-answer check:** MD said delete a row / db_datareader ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/database-level-roles

---

### Q123. Contoso: tiered table and row access
<sub>ID U-135</sub>

**Sources:** MD-082   **Type:** MCQ   **Case study:** Contoso

**Question:**
(Contoso case study: each customer gets its own Azure SQL database with a complete copy of the financial data. The data a customer can access depends on the service agreement tier, and customers can change tiers. Business requirement: "Provide the customers with different table and row access based on the customer's service agreement.")
You are evaluating the business goals. Which feature should you use to provide customers with the required level of access based on their service agreement?

**Options:**
A. dynamic data masking
B. Conditional Access in Azure
C. service principals
D. row-level security (RLS)

**✅ Answer:** D

**Why:** The requirement is to limit which rows (and tables) each customer can see in a database that holds the full data set. RLS filters rows through a security policy and predicate function. Table-level access is handled by ordinary object permissions alongside it. Dynamic data masking only obfuscates column values in query results and doesn't restrict rows. Conditional Access controls sign-in conditions, and service principals are identities, so neither restricts data within the database.

**Source-answer check:** MD said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/row-level-security

---

### Q124. RLS on Orders: policy plus TVF
<sub>ID U-136</sub>

**Sources:** MD-107, NOTE-024   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure SQL database named DB1 that contains a table named Orders. The Orders table contains a row for each sales order. Each sales order includes the name of the user who placed the order. You need to implement row-level security (RLS). The solution must ensure that the users can view only their respective sales orders. What should you include in the solution?

**Options:**
Box 1 – Create: A materialized view in DB1 / A security policy in the Orders table / Database scoped credentials in DB1
Box 2 – Control access to the rows by using: A masking rule / A table-valued function / The CONTAINS predicate

**✅ Answer:** Box 1: A security policy in the Orders table; Box 2: A table-valued function

**Why:** RLS is implemented with CREATE SECURITY POLICY, which adds a filter predicate to the Orders table. The predicate is an inline table-valued function that compares the row's user name with USER_NAME() or SESSION_CONTEXT. A masking rule is dynamic data masking, which hides column values but never removes rows. CONTAINS is a full-text search predicate. Materialized views and database scoped credentials have nothing to do with RLS.

**Source-answer check:** MD said security policy / table-valued function ✔. NOTE: Box 1 security policy ✔. The dump's grey highlight on "A masking rule" for Box 2 is wrong ✘; the faint hand tick on "A table-valued function" is correct ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/row-level-security

---

### Q125. Multi-company RLS in dedicated SQL pool
<sub>ID U-137</sub>

**Sources:** CQ-076, MD-096   **Type:** MultiSelect   **Case study:** —

**Question:**
You are designing a security model for an Azure Synapse Analytics dedicated SQL pool that will support multiple companies. You need to ensure that users from each company can view only the data of their respective company. Which two objects should you include in the solution? Each correct answer presents part of the solution.

**Options:**
A. a column encryption key
B. asymmetric keys
C. a function
D. a custom role-based access control (RBAC) role
E. a security policy

**✅ Answer:** C, E

**Why:** This is row-level security. You create an inline table-valued function as the filter predicate, for example matching the row's CompanyID to the user's company, and bind it to the table with CREATE SECURITY POLICY. Column encryption keys (Always Encrypted) and asymmetric keys handle encryption, not row filtering. Azure RBAC roles control management-plane and workspace access, not rows inside a table. *Scope note: the question is set in Synapse, but RLS works the same way in Azure SQL Database and Azure SQL Managed Instance, which is the in-scope DP-300 skill.*

**Source-answer check:** CQ said C, E ✔. MD lacked the option list, but its explanation (security policy + inline TVF) matches ✔. Options were taken from CQ.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/row-level-security

---

### Q126. New-AzSqlServer: identity plus CMK key
<sub>ID U-138</sub>

**Sources:** MD-121   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription. You need to deploy a logical SQL server by using PowerShell. The solution must ensure that the logical SQL server can create Microsoft Entra users and provide Transparent Data Encryption (TDE) with a customer-managed key. How should you complete the command?

```powershell
New-AzSqlServer -ResourceGroupName "RG1" -ServerName "SQL1" -Location "EastUS" -ErrorAction Stop `
  -Tags @{Environment="Databases";"Department"="Data Tech"} [Box 1] [Box 2] `
  "https://db1.vault.azure.net/keys/dbkey/01234234512345678901234561823942"
```

**Options:**
Box 1: -AssignIdentity / -FederatedClientId / -KeyId
Box 2: -AssignIdentity / -FederatedClientId / -KeyId

**✅ Answer:** Box 1: -AssignIdentity; Box 2: -KeyId

**Why:** -AssignIdentity creates and assigns a Microsoft Entra (managed) identity to the server. The server needs that identity to reach Key Vault for the TDE protector, and to query Microsoft Entra ID when creating Entra users (with Directory Readers / Graph permissions). -KeyId takes the Key Vault key URI that follows it, so Box 2 must be -KeyId. -FederatedClientId is only for cross-tenant CMK ("do not set this value if you do not intend to use Cross-Tenant CMK"), and nothing in the question is cross-tenant. The cmdlet doc's own CMK example uses `-AssignIdentity ... -KeyId "<vault key URI>"`.

**Source-answer check:** MD conflicts with itself. Its explanation text says -AssignIdentity / -KeyId ✔; its answer image highlights -FederatedClientId / -KeyId ✘ (FederatedClientId is cross-tenant only).
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/powershell/module/az.sql/new-azsqlserver

---

### Q127. TDE on SQL VM: create DMK first
<sub>ID U-139</sub>

**Sources:** MD-139   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure virtual machine that hosts a Microsoft SQL Server 2022 database named DB1. You need to enable Transparent Data Encryption (TDE) for DB1. What should you create first?

**Options:**
A. a database scoped credential
B. a database encryption key (DEK)
C. an X.509 certificate
D. a database master key (DMK)

**✅ Answer:** D

**Why:** On SQL Server (including on an Azure VM), TDE is set up in this order: (1) create a master key in master, (2) create a certificate protected by that master key, (3) create the DEK in the user database, protected by the certificate, (4) ALTER DATABASE … SET ENCRYPTION ON. The certificate and DEK depend on the DMK, so they come later. A database scoped credential isn't part of TDE.

**Source-answer check:** MD said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/transparent-data-encryption

---

### Q128. TDE on SQL VM: object sequence
<sub>ID U-140</sub>

**Sources:** MD-125   **Type:** DragDrop   **Case study:** —

**Question:**
You have an instance of SQL Server on Azure Virtual Machines named SQL1. SQL1 contains a database named DB1. You need to enable Transparent Data Encryption (TDE) for DB1. Which three objects should you create in sequence? To answer, move the appropriate objects from the list of objects to the answer area and arrange them in the correct order.

**Options:**
- a database encryption key in the master database
- a master key in the master database
- a certificate in DB1
- a certificate in the master database
- a master key in DB1
- a database encryption key in DB1

**✅ Answer:** 1 → a master key in the master database; 2 → a certificate in the master database; 3 → a database encryption key in DB1

**Why:** Per the TDE docs, the DMK and the certificate must be stored in master, while the DEK is created and stored in the user database (`USE DB1; CREATE DATABASE ENCRYPTION KEY … ENCRYPTION BY SERVER CERTIFICATE …`), followed by `ALTER DATABASE DB1 SET ENCRYPTION ON`. A certificate or master key in DB1 can't protect the DEK, and the DEK can't live in master.

**Source-answer check:** MD said master key (master) → certificate (master) → DEK (DB1) ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/transparent-data-encryption

---

### Q129. TDE with Key Vault EKM sequence
<sub>ID U-141</sub>

**Sources:** MD-123   **Type:** DragDrop   **Case study:** —

**Question:**
You have an Azure key vault named Vault1 and a SQL Server on Azure Virtual Machines instance named SQL1. SQL1 hosts a database named DB1. You need to configure Transparent Data Encryption (TDE) on DB1 to use a key in Vault1. Which four actions should you perform in sequence?

**Options:**
- On SQL1, create a cryptographic provider and a Microsoft SQL Server credential.
- On SQL1, create a symmetric key.
- On SQL1, create a login from the asymmetric key.
- On SQL1, create a login from the symmetric key.
- Create a Microsoft Entra service principal and grant the service principal permissions for Vault1.
- On SQL1, create an asymmetric key.

**✅ Answer:**
1. Create a Microsoft Entra service principal and grant the service principal permissions for Vault1.
2. On SQL1, create a cryptographic provider and a Microsoft SQL Server credential.
3. On SQL1, create an asymmetric key.
4. On SQL1, create a login from the asymmetric key.

**Why:** The documented EKM flow is: (1) app registration / service principal with key wrap and unwrap rights on the vault; (2) `CREATE CRYPTOGRAPHIC PROVIDER … FROM FILE = '…Microsoft.AzureKeyVaultService.EKM.dll'` and `CREATE CREDENTIAL … FOR CRYPTOGRAPHIC PROVIDER`; (3) `CREATE ASYMMETRIC KEY … FROM PROVIDER [AzureKeyVault_EKM] WITH PROVIDER_KEY_NAME = …`; (4) `CREATE LOGIN TDE_Login FROM ASYMMETRIC KEY …`, mapped to the credential. The DEK is then created `ENCRYPTION BY SERVER ASYMMETRIC KEY`. The asymmetric key is opened *from the provider*, so the provider must exist first. EKM supports only RSA (asymmetric) keys, so the symmetric-key options are distractors. (SQL Server 2022 CU17+ on an Azure VM can use a managed identity instead of a service principal, but the order is the same.)

**Source-answer check:** MD gave service principal → asymmetric key → provider + credential → login ✘. Steps 2 and 3 are swapped: CREATE ASYMMETRIC KEY … FROM PROVIDER fails if the cryptographic provider doesn't exist yet.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/setup-steps-for-extensible-key-management-using-the-azure-key-vault

---

### Q130. Encrypt Synapse data warehouse at rest
<sub>ID U-142</sub>

**Sources:** MD-095, CQ-075   **Type:** MCQ   **Case study:** —

**Question:**
You have a data warehouse in Azure Synapse Analytics. You need to ensure that the data in the data warehouse is encrypted at rest. What should you enable?

**Options:**
A. Transparent Data Encryption (TDE)
B. Advanced Data Security for this database
C. Always Encrypted for all columns
D. Secure transfer required

**✅ Answer:** A

**Why:** TDE encrypts data and log files (and backups) at rest for SQL Server, Azure SQL Database, Azure SQL Managed Instance and Azure Synapse dedicated SQL pools. Advanced Data Security (now Microsoft Defender for SQL) covers vulnerability assessment and threat detection, not encryption. Always Encrypted is column-level client-side encryption, not the way to encrypt the whole warehouse at rest. "Secure transfer required" is a storage-account setting for encryption in transit. *Scope note: the question is set in Synapse, but TDE works the same way in Azure SQL Database and Managed Instance, which is the in-scope DP-300 skill. In Azure SQL, TDE is on by default for new databases.*

**Source-answer check:** MD and CQ said A ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/transparent-data-encryption

---


## D3 — Monitor, configure, and optimize database resources (20–25%)

*59 questions*

### Q131. Automatic tuning on Managed Instance
<sub>ID U-143</sub>

**Sources:** MD-190   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL managed instance named MI1. You need to implement automatic tuning for the databases of MI1. What should you do?

**Options:**
A. Use the REST API to call the patch operation and modify the AutomaticTuningServerMode property.
B. From the Azure portal, configure automatic tuning.
C. Use Transact-SQL to enable the FORCE_LAST_GOOD_PLAN option.

**✅ Answer:** C

**Why:** SQL Managed Instance supports only the FORCE LAST GOOD PLAN automatic tuning option, and the docs say it can only be configured through T-SQL (`ALTER DATABASE … SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON)`). Portal and REST API configuration (server automatic tuning mode) apply only to Azure SQL Database logical servers.

**Source-answer check:** MD said C ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/automatic-tuning-enable ; https://learn.microsoft.com/en-us/azure/azure-sql/database/automatic-tuning-overview

---

### Q132. SIM: disable FORCE_LAST_GOOD_PLAN on db1
<sub>ID U-144</sub>

**Sources:** MD-049   **Type:** Simulation (lab)   **Case study:** —

**Question:**
SIMULATION. You plan to perform performance testing of db1. You need to prevent db1 from reverting to the last known good query plan. To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:** none (lab task)

**✅ Answer:** Turn off the FORCE_LAST_GOOD_PLAN (automatic plan correction) option for db1. It is on by default through the Azure defaults that db1 inherits.
- **T-SQL (SSMS, connected to db1):**
```sql
ALTER DATABASE [db1] SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = OFF);
-- verify
SELECT name, desired_state_desc, actual_state_desc
FROM sys.database_automatic_tuning_options;
```
- **Portal alternative:** Azure portal > SQL databases > db1 > Intelligent Performance > **Automatic tuning** > set **Force plan** to **OFF** (instead of Inherit) > **Apply**.

**Why:** The Azure defaults enable FORCE_LAST_GOOD_PLAN, which forces the previous good plan whenever it detects a plan regression. Setting the option explicitly to OFF at database level overrides the inherited server or Azure default, so the test sees the real plan behavior. Changing the server-level setting would also affect other databases.

**Source-answer check:** MD said FORCE_LAST_GOOD_PLAN = OFF (T-SQL or portal) ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/automatic-tuning-enable

---

### Q133. SQL VM: Query Store then FORCE_LAST_GOOD_PLAN
<sub>ID U-145</sub>

**Sources:** NOTE-212   **Type:** Hotspot   **Case study:** —

**Question:**
You have SQL Server on an Azure virtual machine that contains a database named Db1. You need to enable automatic tuning for Db1. How should you complete the statements?

```sql
ALTER DATABASE [Db1]
  [Box 1]
GO
ALTER DATABASE [Db1]
  [Box 2]
```

**Options:** (same list for Box 1 and Box 2)
- SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = OFF)
- SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON)
- SET AUTOMATIC_TUNING = AUTO
- SET QUERY_STORE = OFF
- SET QUERY_STORE = ON (OPERATION_MODE = READ_ONLY)
- SET QUERY_STORE = ON (OPERATION_MODE = READ_WRITE)

**✅ Answer:** Box 1: SET QUERY_STORE = ON (OPERATION_MODE = READ_WRITE); Box 2: SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON)

**Why:** Automatic tuning in SQL Server (2017+), including on an Azure VM, means automatic plan correction, which depends on Query Store. `FORCE_LAST_GOOD_PLAN = ON` "fails if the Query Store isn't enabled or if the Query Store isn't in Read-Write mode", so Query Store must be enabled in READ_WRITE first. For SQL Server, the ALTER DATABASE syntax is only `AUTOMATIC_TUNING ( FORCE_LAST_GOOD_PLAN = { DEFAULT | ON | OFF } )`. The `AUTOMATIC_TUNING = AUTO | INHERIT | CUSTOM` form exists only in Azure SQL Database (logical server), which doesn't apply to a SQL Server VM. READ_ONLY Query Store and FORCE_LAST_GOOD_PLAN = OFF don't enable anything.

**Source-answer check:** NOTE marked Box 1 QUERY_STORE READ_WRITE ✔ and Box 2 `SET AUTOMATIC_TUNING = AUTO` ✘. AUTO is Azure SQL Database-only syntax; on SQL Server the correct statement is FORCE_LAST_GOOD_PLAN = ON.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-set-options?view=sql-server-ver16 ; https://learn.microsoft.com/en-us/azure/azure-sql/database/automatic-tuning-enable

---

### Q134. Stop reads blocking writes: RCSI
<sub>ID U-146</sub>

**Sources:** NOTE-27-img022   **Type:** MCQ   **Case study:** —

**Question:**
You deploy a database to an Azure SQL Database managed instance. You need to prevent read queries from blocking queries that are trying to write to the database. Which database option should you set?

**Options:**
A. PARAMETERIZATION to FORCED
B. PARAMETERIZATION to SIMPLE
C. Delayed Durability to Forced
D. READ_COMMITTED_SNAPSHOT to ON

**✅ Answer:** D

**Why:** With READ_COMMITTED_SNAPSHOT ON, read-committed readers use row versions in tempdb (or the persistent version store under ADR) instead of shared locks, so readers don't block writers and writers don't block readers (`ALTER DATABASE … SET READ_COMMITTED_SNAPSHOT ON`). PARAMETERIZATION FORCED or SIMPLE only changes plan caching and reuse. Delayed durability reduces log-flush latency on commit but doesn't change reader/writer locking. (RCSI is on by default in Azure SQL Database; on a managed instance, check `sys.databases.is_read_committed_snapshot_on` and set it explicitly.)

**Source-answer check:** NOTE marked D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/t-sql/statements/set-transaction-isolation-level-transact-sql

---

### Q135. Change MAXDOP on Azure SQL Database
<sub>ID U-147</sub>

**Sources:** MD-177   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database named db1 on a server named server1. You need to modify the MAXDOP settings for db1. What should you do?

**Options:**
A. Connect to db1 and run the sp_configure command.
B. Connect to the master database of server1 and run the sp_configure command.
C. Configure the extended properties of db1.
D. Modify the database scoped configuration of db1.

**✅ Answer:** D

**Why:** In Azure SQL Database, MAXDOP is set per database with `ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP = n;` while connected to db1. Server-level `sp_configure 'max degree of parallelism'` isn't available on an Azure SQL Database logical server, so neither sp_configure option works. Extended properties are only metadata annotations.

**Source-answer check:** MD said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql

---

### Q136. SIM: set compatibility level 110
<sub>ID U-148</sub>

**Sources:** MD-050   **Type:** Simulation (lab)   **Case study:** —

**Question:**
SIMULATION. You have a legacy application written for Microsoft SQL Server 2012. The application will be the only application that accesses db1. You need to ensure that db1 is compatible with all the features and syntax of SQL Server 2012. To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:** none (lab task)

**✅ Answer:** Set db1's database compatibility level to **110** (SQL Server 2012).
- **T-SQL (SSMS, connected to the logical server, query window on db1 or master):**
```sql
ALTER DATABASE [db1] SET COMPATIBILITY_LEVEL = 110;
-- verify
SELECT name, compatibility_level FROM sys.databases WHERE name = 'db1';
```
- **SSMS GUI alternative:** Object Explorer > right-click db1 > Properties > **Options** page > Compatibility level = **SQL Server 2012 (110)** > OK.

**Why:** Compatibility level controls the T-SQL and query-processing behavior of a database. Azure SQL Database supports levels 100–170, so 110 is valid. Because only this legacy app uses db1, lowering the level has no side effects on other apps. (Discontinued features aren't restored by compatibility level, but 110 matches SQL Server 2012 behavior.) Don't use LEGACY_CARDINALITY_ESTIMATION alone: it only affects the cardinality estimator, not syntax or features.

**Source-answer check:** MD said compatibility level 110 via SSMS Options or ALTER DATABASE ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-compatibility-level

---

### Q137. SIM: QUERY_OPTIMIZER_HOTFIXES on db1, db2
<sub>ID U-149</sub>

**Sources:** MD-047   **Type:** Simulation (lab)   **Case study:** —

**Question:**
SIMULATION. You need to ensure that any enhancements made to the Query Optimizer through patches are available to db1 and db2 on sql12345678. To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:** none (lab task)

**✅ Answer:** Enable the QUERY_OPTIMIZER_HOTFIXES database scoped configuration in **each** database:
1. In SSMS, connect to sql12345678.database.windows.net.
2. Open a query window **on db1** and run:
```sql
ALTER DATABASE SCOPED CONFIGURATION SET QUERY_OPTIMIZER_HOTFIXES = ON;
```
3. Open a query window **on db2** (Azure SQL Database doesn't support `USE` to switch databases) and run the same statement.
4. Optional check in each database: `SELECT name, value FROM sys.database_scoped_configurations WHERE name = 'QUERY_OPTIMIZER_HOTFIXES';`
(GUI alternative: database Properties > Options > Database Scoped Configurations > Query Optimizer Fixes = ON.)

**Why:** QUERY_OPTIMIZER_HOTFIXES = ON enables query optimizer hotfixes released after the highest compatibility level, whatever the database's compatibility level. It is equivalent to trace flag 4199, which you can't set on Azure SQL Database. It is a database-scoped setting, so it must be set in both db1 and db2.

**Source-answer check:** MD said ALTER DATABASE SCOPED CONFIGURATION QUERY_OPTIMIZER_HOTFIXES = ON in each DB ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql

---

### Q138. Single-use plans: OPTIMIZE_FOR_AD_HOC_WORKLOADS
<sub>ID U-150</sub>

**Sources:** NOTE-004   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database. You discover that the plan cache is full of compiled plans that were used only once. You run the `SELECT * FROM sys.database_scoped_configurations` Transact-SQL command and receive the results shown in the following table.

| configuration_id | name | value | is_value_default |
|---|---|---|---|
| 1 | LEGACY_CARDINALITY_ESTIMATION | 0 | 1 |
| 2 | QUERY_OPTIMIZER_HOTFIXES | 0 | 1 |
| 3 | OPTIMIZE_FOR_AD_HOC_WORKLOADS | 0 | 1 |
| 4 | ACCELERATED_PLAN_FORCING | 1 | 1 |

You need to relieve the memory pressure. What should you configure?

**Options:**
A. LEGACY_CARDINALITY_ESTIMATION
B. QUERY_OPTIMIZER_HOTFIXES
C. OPTIMIZE_FOR_AD_HOC_WORKLOADS
D. ACCELERATED_PLAN_FORCING

**✅ Answer:** C

**Why:** With `OPTIMIZE_FOR_AD_HOC_WORKLOADS = ON`, the first compile of a batch caches only a small compiled-plan stub. The full plan is cached only if the batch runs again, so single-use plans stop filling the plan cache. A and B change optimizer behavior, not cache size. D (accelerated plan forcing) is about forced plans in Query Store.

**Source-answer check:** Note marked C ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql ; https://learn.microsoft.com/en-us/archive/blogs/sqlserverstorageengine/database-scoped-optimizing-for-ad-hoc-workloads

---

### Q139. Estimated plan in portal query editor
<sub>ID U-151</sub>

**Sources:** NOTE-231   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database named DB1. You need to display the estimated execution plan of a query by using the query editor in the Azure portal. What should you do first?

**Options:**
A. Run the SET SHOWPLAN_ALL Transact-SQL statement.
B. For DB1, set QUERY_CAPTURE_MODE of Query Store to All.
C. Run the SET FORCEPLAN Transact-SQL statement.
D. Enable Query Store for DB1.

**✅ Answer:** A

**Why:** The Azure portal query editor has no graphical "display estimated plan" button. It only runs T-SQL and shows result sets. Running `SET SHOWPLAN_ALL ON` first makes later statements return their estimated plan as rows without executing. Microsoft Learn's DP-300 content says: "To see a text-based estimated plan, run the command SET SHOWPLAN_ALL ON before executing the query." Query Store (B, D) records the plans of queries that have already run and isn't needed for an estimated plan. It is also on by default in Azure SQL Database. SET FORCEPLAN (C) only forces the join order.

**Source-answer check:** Note marked A ✔. Some dump-discussion sites (pass4success) have a crowd vote for D ✘, with no doc support.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/training/modules/explore-query-performance-optimization/2-understand-query-plans ; https://learn.microsoft.com/en-us/azure/azure-sql/database/query-editor

---

### Q140. Identify plan type and operator to tune
<sub>ID U-152</sub>

**Sources:** NOTE-img020   **Type:** Hotspot   **Case study:** —

**Question:**
*(Stem intro is cut off in the source and no full version was found. Reconstructed: you run a query against WideWorldImporters and see the following exhibit.)*

Exhibit (graphical plan in SSMS):
- Header: **"Estimated query progress: 100%"**. Query 1: Query cost (relative to the batch): 100%
- `select OrderID, PickedbyPersonID, OrderDate from WideWorldImporters.Sales.Orders where CustomerID = 120`
- Missing Index (Impact 99.6573): CREATE NONCLUSTERED INDEX … ON [Sales].[Orders] …
- SELECT (0.126s) ← Nested Loops (Inner Join) 0.126s, 392 of 446 (87%)
  - Index Seek (NonClustered) [Orders].[FK_Sales_Orders_CustomerID]: 0.126s, 392 of 446 (87%)
  - Key Lookup (Clustered) [Orders].[PK_Sales_Orders]: 0.126s, 392 of 446 (87%)

Use the drop-down menus to select the answer choice that completes each statement based on the information presented in the graphic. NOTE: Each correct selection is worth one point.

**Options:**
- Statement 1: The exhibit shows [an actual execution plan | an estimated execution plan | Live Query Statistics].
- Statement 2: The [Index Seek | Key Lookup | Nested Loops] operator in the execution plan indicates that the query would benefit from performance tuning.

**✅ Answer:** Statement 1: **Live Query Statistics**; Statement 2: **Key Lookup**

**Why:** The overall "Estimated query progress: N%" banner and the per-operator "elapsed time + actual of estimated rows (percent)" display belong to the SSMS Live Query Statistics view, which "displays the overall query progress and operator-level run-time execution statistics". An estimated plan has no runtime figures. A completed actual plan has no query-progress banner. The Key Lookup back to the clustered PK, together with the missing-index hint, shows the nonclustered index doesn't cover the query. Adding the selected columns as INCLUDE columns would remove the lookup.

**Source-answer check:** Note marked Statement 1 = "an actual execution plan" ✘ (the progress banner points to Live Query Statistics). Statement 2 = Key Lookup ✔.
**Confidence:** Medium (stem cut off; the call on Statement 1 rests on the "Estimated query progress" banner in the exhibit)   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/performance/live-query-statistics ; https://learn.microsoft.com/en-us/training/modules/explore-query-performance-optimization/2-understand-query-plans

---

### Q141. Key Lookup fix: include columns
<sub>ID U-153</sub>

**Sources:** NOTE-img058   **Type:** Hotspot   **Case study:** —

**Question:**
*(Stem intro is cut off in the source. Reconstructed: you have a query with the execution plan shown in the exhibit.)*

Exhibit:
`SELECT [SalesOrderID],[SalesOrderDetailID],[OrderQty],[ProductID],[UnitPrice] FROM [Sales].[SalesOrderDetail] WHERE [ModifiedDate]=@1`
Plan: SELECT (Cost 0%) ← Nested Loops (Inner Join) (Cost 0%, 148 of 148) ← [Index Seek (NonClustered) [SalesOrderDetail].[NCI_SalesOrderDetail_M…] Cost 1%] + [Key Lookup (Clustered) [SalesOrderDetail].[PK_SalesOrderDetail_Sal…] Cost 99%]

Use the drop-down menus to select the answer choice that completes each statement based on the information presented in the graphic. NOTE: Each correct selection is worth one point.

**Options:**
- Statement 1: The performance issue stems from the [Select | Index Seek | Key Lookup | Nested Loops] operator.
- Statement 2: The performance issue can be resolved by adding include columns to the [heap | clustered index | nonclustered index].

**✅ Answer:** Statement 1: **Key Lookup**; Statement 2: **nonclustered index**

**Why:** The Key Lookup takes 99% of the cost. It fetches OrderQty, ProductID and UnitPrice from the clustered index for every row the nonclustered seek on ModifiedDate finds. Adding those columns as INCLUDE columns to the nonclustered index makes it covering and removes the lookup. You can't add INCLUDE columns to a clustered index or a heap.

**Source-answer check:** Note marked Key Lookup / nonclustered index ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/training/modules/explore-query-performance-optimization/2-understand-query-plans

---

### Q142. Last actual plan parameter values
<sub>ID U-154</sub>

**Sources:** NOTE-045   **Type:** MultiSelect   **Case study:** —

**Question:**
You have SQL Server on an Azure virtual machine that contains a database named DB1. You have an application that queries DB1 to generate a sales report. You need to see the parameter values from the last time the query was executed. Which two actions should you perform? Each correct answer presents part of the solution. NOTE: Each correct selection is worth one point.

**Options:**
A. Enable Last_Query_Plan_Stats in the master database
B. Enable Lightweight_Query_Profiling in DB1
C. Enable Last_Query_Plan_Stats in DB1
D. Enable Lightweight_Query_Profiling in the master database
E. Enable PARAMETER_SNIFFING in DB1

**✅ Answer:** B, C

**Why:** `sys.dm_exec_query_plan_stats` returns the last known actual plan, including runtime parameter values. It is opt-in per database through `ALTER DATABASE SCOPED CONFIGURATION SET LAST_QUERY_PLAN_STATS = ON`, run in DB1. It also "works under the lightweight query execution statistics profiling infrastructure", the database-scoped LIGHTWEIGHT_QUERY_PROFILING option in DB1. Database-scoped configurations apply to the database you set them in, not to master (A, D). PARAMETER_SNIFFING controls plan compilation, not plan capture (E).

**Source-answer check:** Note marked B, C ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-plan-stats-transact-sql ; https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql

---

### Q143. ADatum: event_file target for DB3
<sub>ID U-155</sub>

**Sources:** MD-148   **Type:** MCQ   **Case study:** ADatum v1 (financial services)

**Question:**
*(Case study: ADatum v1. DB3 is currently monitored with Extended Events whose event_file target is a file share on a local disk of SVR3. DB3 will be migrated to Azure SQL Database. Management requirement: "Continue using Extended Events to monitor DB3.")*
You need to identify the event_file target for monitoring DB3 after the migration to Azure SQL Database. The solution must meet the management requirements. What should you use as the event_file target?

**Options:**
A. a SQL Server filegroup
B. an Azure SQL database
C. an Azure Files share
D. an Azure Blob Storage container

**✅ Answer:** D

**Why:** In Azure SQL Database (and Managed Instance), the event_file target supports only blobs in Azure Storage as the `filename` value, given as `https://<account>.blob.core.windows.net/<container>/<file>.xel` with a matching database-scoped credential. Local disks, file shares (Azure Files), filegroups and databases aren't valid event_file destinations.

**Source-answer check:** MD said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server

---

### Q144. Correlate XE session with ETW
<sub>ID U-156</sub>

**Sources:** MD-206   **Type:** MCQ   **Case study:** —

**Question:**
You have an instance of SQL Server on Azure Virtual Machines named SQL1. SQL1 contains an Extended Events session named session1 that captures Microsoft SQL Server events. You need to correlate the session events with events captured by Event Tracing for Windows (ETW). What should you do for session1?

**Options:**
A. Modify the Set Session Event Filters settings.
B. Add a target.
C. Add an action.
D. Modify the Specify Session Data Storage settings.

**✅ Answer:** B

**Why:** Extended Events interoperates with ETW through the Event Tracing for Windows target, `package0.etw_classic_sync_target`. Adding that target (`ALTER EVENT SESSION session1 ON SERVER ADD TARGET package0.etw_classic_sync_target`) sends the session's events to ETW, where you can correlate them with OS and application ETW events. This target is available only in SQL Server, which is why the question uses a VM. Filters and actions change what is captured, not where it goes. D refers to the target page of the New Session wizard, but the defined action is to add a target.

**Source-answer check:** MD said B ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server

---

### Q145. XE raw data to Storage, minimal impact
<sub>ID U-157</sub>

**Sources:** MD-202   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription that contains an instance of SQL Server on Azure Virtual Machines. The virtual machine hosts a database named DB1. You need to monitor DB1 by using Extended Events. The solution must meet the following requirements:
- Capture raw event data and store the data in Azure Storage.
- Minimize the performance impact of capturing extended events.

How should you complete the Transact-SQL statement? To answer, select the appropriate options in the answer area. NOTE: Each correct selection is worth one point.
```sql
CREATE EVENT SESSION session1 ON DATABASE
  ADD EVENT sqlserver.sql_statement_starting
    (
    ACTION (sqlserver.sql_text)
    WHERE statement LIKE 'UPDATE gmTabEmployee%'
    )
  ADD TARGET
    package0.[Box1]
      (
      SET filename = 'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent/anyfilenamexel242b.xel'
      )
    WITH
      (MAX_MEMORY = 10 MB,
      EVENT_RETENTION_MODE = [Box2],
  MAX_DISPATCH_LATENCY = 3 SECONDS)
```

**Options:**
- Box1: event_file | event_stream | ring_buffer
- Box2: ALLOW_MULTIPLE_EVENT_LOSS | ALLOW_SINGLE_EVENT_LOSS | NO_EVENT_LOSS

**✅ Answer:** Box1: **event_file**; Box2: **ALLOW_MULTIPLE_EVENT_LOSS**

**Why:** event_file is the only listed target that writes raw event data to persistent storage, including a blob in Azure Storage (the `filename` URL). ring_buffer keeps data in memory only. event_stream is for .NET consumers and can't be added through T-SQL. ALLOW_MULTIPLE_EVENT_LOSS lets whole buffers be dropped under pressure and has the lowest performance impact. ALLOW_SINGLE_EVENT_LOSS is the default middle ground. NO_EVENT_LOSS can cause noticeable performance degradation because sessions can stall waiting for buffers to flush.

**Source-answer check:** MD said event_file / ALLOW_MULTIPLE_EVENT_LOSS ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server ; https://learn.microsoft.com/sv-se/sql/t-sql/statements/alter-event-session-transact-sql?view=sql-server-2017

---

### Q146. SIM: capture queries and users to storage
<sub>ID U-158</sub>

**Sources:** MD-223   **Type:** Simulation   **Case study:** —

**Question:**
SIMULATION. You need to capture the following information for all the databases on sq12345678:
- Queries to the databases
- Users that executed the queries

The captured information must be stored in sa12345678. To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:** none (lab task)

**✅ Answer (recommended, if sq12345678 is the Azure SQL logical server, as in the DP-300 lab):** Enable **server-level Azure SQL Auditing** to storage account sa12345678.
1. In the Azure portal, open the SQL server **sq12345678** → **Security** → **Auditing**.
2. Set **Enable Azure SQL Auditing** to On. This is server-level, so it applies to every existing and new database on the server.
3. Select the **Storage** destination → choose subscription → storage account **sa12345678**. Set retention if you want it. Choose storage access keys or managed identity for authentication, whichever matches the storage account configuration.
4. Select **Save**. The default audit action groups (BATCH_COMPLETED_GROUP, SUCCESSFUL_DATABASE_AUTHENTICATION_GROUP, FAILED_DATABASE_AUTHENTICATION_GROUP) record each completed statement's text (`statement`) and the principal that ran it (`server_principal_name` / `database_principal_name`).

**Alternative (source's method; valid if sq12345678 is a SQL Server instance on the VM or a managed instance):** In SSMS → **Management** → **Extended Events** → **Sessions** → **New Session**. Add the `sql_statement_completed` / `sql_batch_completed` events with the actions `sql_text`, `username` / `server_principal_name` and `database_name`. On **Data Storage**, choose **event_file** with the URL `https://sa12345678.blob.core.windows.net/<container>/<name>.xel`. This needs a credential named after the container URL (SAS or managed identity). Then start the session. On an Azure SQL logical server, XE sessions are database-scoped (`ON DATABASE`) and appear under each database, not under Management. You would need one session per database, so auditing is the practical "all databases" answer.

**Why:** Server-level auditing is the built-in way to record, for all databases on a logical server, which statements ran and who ran them, written as .xel files to a storage account. Extended Events with an event_file blob target also satisfies the requirement, but only at server scope on SQL Server or MI.

**Source-answer check:** MD described the SSMS Extended Events (Management > Extended Events) method ⚠. That works for a SQL Server or MI instance, but on an Azure SQL logical server it can't cover "all databases" from one session. Auditing is the better or expected method there.
**Confidence:** Medium (the lab environment's server type is not stated; both methods meet the requirement where they apply)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/auditing-overview ; https://learn.microsoft.com/en-in/training/modules/configure-azure-sql-auditing/2-describe-azure-sql-auditing-capabilities ; https://learn.microsoft.com/en-us/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server

---

### Q147. Query index fragmentation information
<sub>ID U-159</sub>

**Sources:** MD-203   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database named DB1. You need to query the fragmentation information of data and indexes for the tables in DB1. Which command should you use?

**Options:**
A. sys.dm_db_index_usage_stats
B. DBCC CHECKALLOC
C. DBCC SHOWCONTIG
D. sys.dm_db_index_physical_stats

**✅ Answer:** D

**Why:** `sys.dm_db_index_physical_stats` "returns size and fragmentation information for the data and indexes of a specified table or view" (`avg_fragmentation_in_percent`, page density), and it is supported in Azure SQL Database. DBCC SHOWCONTIG is deprecated and replaced by this DMF. index_usage_stats shows seeks, scans and updates, not fragmentation. CHECKALLOC checks allocation consistency.

**Source-answer check:** MD said D ✔
**Confidence:** High   **Ref:** https://msdn2.microsoft.com/en-us/library/ms188917.aspx (sys.dm_db_index_physical_stats on Learn)

---

### Q148. Intelligent Insights missing indexes: automatic tuning
<sub>ID U-160</sub>

**Sources:** MD-180, NOTE-072   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database named db1 on a server named server1. The Intelligent Insights diagnostics log identifies that several tables are missing indexes. You need to ensure that indexes are created for the tables. What should you do?

**Options:**
A. Run the DBCC SQLPERF command.
B. Run the DBCC DBREINDEX command.
C. Modify the automatic tuning settings for db1.
D. Modify the Query Store settings for db1.

**✅ Answer:** C

**Why:** For the "Missing index" pattern, Intelligent Insights recommends enabling automatic tuning, which "automatically tunes and creates indexes". Turn on the CREATE INDEX automatic tuning option for db1 (`ALTER DATABASE CURRENT SET AUTOMATIC_TUNING (CREATE_INDEX = ON)`). It's off by default, since the Azure defaults enable only FORCE_LAST_GOOD_PLAN. DBCC DBREINDEX rebuilds existing indexes and DBCC SQLPERF reports log space; neither creates new indexes. Query Store settings only control capture.

**Source-answer check:** MD & Note said C ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/intelligent-insights-troubleshoot-performance ; https://learn.microsoft.com/en-us/azure/azure-sql/database/automatic-tuning-overview

---

### Q149. DMV for operations on an index
<sub>ID U-161</sub>

**Sources:** MD-200   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database named DB1 that contains a nonclustered index named index1. End users report slow queries when they use index1. You need to identify the operations that are being performed on the index. Which dynamic management view should you use?

**Options:**
A. sys.dm_exec_query_plan_stats
B. sys.dm_db_index_physical_stats
C. sys.dm_db_index_operational_stats
D. sys.dm_db_index_usage_stats

**✅ Answer:** C

**Why:** `sys.dm_db_index_operational_stats` returns the lower-level data access, locking and latching statistics for each partition of an index. Its counters cover the operations performed on the index (leaf insert/update/delete/ghost counts, range scans, singleton lookups, page splits) plus row and page lock waits, latch waits and page I/O latch waits. That is what you need to see what is happening on index1 and why it is slow. physical_stats (B) gives fragmentation and size. usage_stats (D) gives only coarse seek/scan/lookup/update counts and last-used times, with no contention detail. query_plan_stats (A) returns the last actual plan, not index activity.

**Source-answer check:** MD said C ✔. The source misspells option D as "useage"; it's the same DMV.
**Confidence:** Medium (usage_stats also counts "index operations", but operational_stats is the documented tool for operation-level activity and contention)   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-db-index-operational-stats-transact-sql

---

### Q150. SIM: rebuild index with fill factor 70
<sub>ID U-162</sub>

**Sources:** MD-211   **Type:** Simulation   **Case study:** —

**Question:**
SIMULATION. You need to rebuild the index for PK_Address_AddressID in the SalesLT.Address table in db1. The solution must meet the following requirements:
- Ensure that 30 percent of the index page is reserved for future updates and inserts.
- Minimize page splits.

To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:** none (lab task)

**✅ Answer:** Rebuild the index with **FILLFACTOR = 70** (70% full, 30% free).

T-SQL (SSMS or the portal query editor connected to db1):
```sql
ALTER INDEX PK_Address_AddressID ON SalesLT.Address
REBUILD WITH (FILLFACTOR = 70);
```
SSMS GUI alternative:
1. Connect to the server and expand **db1** → **Tables** → **SalesLT.Address** → **Indexes**.
2. Right-click **PK_Address_AddressID** → **Properties** → **Options** page. Set **Fill factor** to **70**. Optionally set **Pad index** to True to apply the same free space to intermediate-level pages.
3. Select **OK**. The index is rebuilt with the new fill factor. Alternatively, right-click → **Rebuild** after you set the property.

**Why:** Fill factor sets the percentage of each leaf page filled when the index is created or rebuilt. 100 − 30% reserved = 70. The free space lets new and updated rows fit without page splits. The setting takes effect only on create or rebuild, so you must REBUILD.

**Source-answer check:** MD said fill factor 70 via index properties ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/sql/relational-databases/indexes/specify-fill-factor-for-an-index

---

### Q151. Faster DBCC CHECKDB on 50-TB DB
<sub>ID U-163</sub>

**Sources:** NOTE-005-img077   **Type:** Hotspot   **Case study:** —

**Question:**
You have a 50-TB Microsoft SQL Server database named DB1. You need to reduce the time it takes to perform database consistency checks of DB1. Which Transact-SQL command should you run? To answer, select the appropriate options in the answer area. NOTE: Each correct selection is worth one point.
```sql
DBCC CHECKDB ([DB1], [Box 1]) WITH [Box 2]
```

**Options:**
- Box 1: NOINDEX | REPAIR_FAST | REPAIR_REBUILD
- Box 2: ALL_ERRORMSGS | NO_INFOMSGS | PHYSICAL_ONLY

**✅ Answer:** Box 1: **NOINDEX**; Box 2: **PHYSICAL_ONLY**

**Why:** NOINDEX skips intensive checks of nonclustered indexes on user tables, and the docs say it "decreases the overall execution time". PHYSICAL_ONLY limits checking to page and record headers and allocation consistency, so it "might cause a much shorter run-time for DBCC CHECKDB on large databases" and is recommended for frequent production use. REPAIR_* options repair instead of speeding checks; they need single-user mode and can't be combined with PHYSICAL_ONLY. ALL_ERRORMSGS and NO_INFOMSGS only change output.

**Source-answer check:** Note marked NOINDEX / PHYSICAL_ONLY ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/t-sql/database-console-commands/dbcc-checkdb-transact-sql

---

### Q152. Recover DB from CHECKSUM error
<sub>ID U-164</sub>

**Sources:** NOTE-3   **Type:** Hotspot   **Case study:** —

**Question:**
You have SQL Server on an Azure virtual machine that contains a database named DB1. The database reports a CHECKSUM error. You need to recover the database. How should you complete the statements? To answer, select the appropriate options in the answer area. NOTE: Each correct selection is worth one point.
```sql
USE master;
ALTER DATABASE [DB1] SET [Box 1] WITH ROLLBACK IMMEDIATE;
GO
DBCC CHECKDB ('DB1', [Box 2]) WITH NO_INFOMSGS;
GO
ALTER DATABASE [DB1] SET [Box 3]
GO
```

**Options:**
- Box 1: OFFLINE | ONLINE | SINGLE_USER | TRUSTWORTHY
- Box 2: NOINDEX | PHYSICAL_ONLY | REPAIR_ALLOW_DATA_LOSS | REPAIR_FAST
- Box 3: MULTI_USER | ONLINE | OPEN | TRUSTWORTHY

**✅ Answer:** Box 1: **SINGLE_USER**; Box 2: **REPAIR_ALLOW_DATA_LOSS**; Box 3: **MULTI_USER**

**Why:** DBCC CHECKDB repair options require the database to be in single-user mode, and `WITH ROLLBACK IMMEDIATE` disconnects other sessions. A CHECKSUM error is page-level physical corruption that only REPAIR_ALLOW_DATA_LOSS can fix, possibly by deallocating pages. REPAIR_FAST is kept for backward compatibility and performs no repairs. NOINDEX and PHYSICAL_ONLY only check. Afterward, set the database back to MULTI_USER. Microsoft's primary recommendation is still to restore from a last known good backup; repair is the last resort.

**Source-answer check:** Note marked SINGLE_USER / REPAIR_ALLOW_DATA_LOSS / MULTI_USER ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/t-sql/database-console-commands/dbcc-checkdb-transact-sql

---

### Q153. Intelligent Insights tempDB contention fix
<sub>ID U-165</sub>

**Sources:** MD-182, NOTE-010, NOTE-008   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database named db1 on a server named server1. The Intelligent Insights diagnostics log identifies queries that cause performance issues due to tempDB contention. You need to resolve the performance issues. What should you do?

**Options:**
A. Implement memory-optimized tables.
B. Run the DBCC FLUSHPROCINDB command.
C. Replace the sequential index keys with nonsequential keys.
D. Run the DBCC DBREINDEX command.

**✅ Answer:** A

**Why:** The Intelligent Insights troubleshooting guide says that to relieve tempdb contention (many concurrent queries creating, using and dropping small temp tables), "You can stop using the temporary tables. You also can use memory-optimized tables." Replacing sequential keys with nonsequential keys (C) is the documented fix for *pagelatch* contention on user indexes, not tempdb. FLUSHPROCINDB clears cached plans and DBREINDEX rebuilds indexes; neither affects tempdb contention.

**Source-answer check:** MD, NOTE-010 & NOTE-008 all said A ✔ (NOTE-010 and NOTE-008 are the same question)
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/intelligent-insights-troubleshoot-performance

---

### Q154. Litware: collect and stream SalesSQLDb1 metrics
<sub>ID U-166</sub>

**Sources:** MD-149, DEMO-3   **Type:** Hotspot   **Case study:** Litware

**Question:**
*(Case study: Litware. SalesSQLDb1 is a geo-replicated Azure SQL database on logical server SalesSrv01A, in the elastic pool SalesSQLDb1Pool. Technical requirement: "All SQL Server and Azure SQL Database metrics related to CPU and storage usage and limits must be analyzed by using Azure built-in functionality.")*
You need to implement the monitoring of SalesSQLDb1. The solution must meet the technical requirements. How should you collect and stream metrics? To answer, select the appropriate options in the answer area. NOTE: Each correct selection is worth one point.

**Options:**
- Collect metrics from: The database only | The elastic pool and the database | The elastic pool only | The server, the elastic pool, and the database
- Stream metrics to: Azure Event Hubs | Azure Log Analytics | Azure Storage

**✅ Answer:** Collect metrics from: **The elastic pool and the database**; Stream metrics to: **Azure Log Analytics**

**Why:** Diagnostic telemetry for pooled databases must be configured on each resource separately. Per the docs: "To configure streaming of diagnostic telemetry for elastic pools and pooled databases, you need to separately configure each resource… The elastic pool container has its own telemetry separate from each individual pooled database's telemetry." The pool's Basic metrics include CPU/eDTU, storage and storage limit. The logical server isn't a diagnostic-telemetry source for these metrics, so "server, pool, and database" is wrong. To *analyze* the metrics with Azure built-in functionality, send them to a Log Analytics workspace (Azure Monitor logs, alerts, workbooks). Event Hubs is for streaming to third-party SIEMs or custom telemetry platforms. Storage is for cheap archiving.

**Source-answer check:** MD & DEMO said "The server, the elastic pool, and the database" + "Azure Event Hubs" ✘. The dump's reasoning, that "Azure SQL Analytics doesn't support SQL on VMs", concerns ManufacturingSQLDb1, not SalesSQLDb1, and Event Hubs offers no built-in analysis. The classic dump key differs from this answer.
**Confidence:** Medium (doc-backed, but contradicts the long-standing dump key)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure

---

### Q155. Litware: DMVs for SalesSQLDb1 blocking
<sub>ID U-167</sub>

**Sources:** DEMO-4   **Type:** MultiSelect   **Case study:** Litware

**Question:**
*(Case study: Litware. "SalesSQLDb1 experiences performance issues that are likely due to out-of-date statistics and frequent blocking queries." SalesSQLDb1 is an Azure SQL database in an elastic pool.)*
You need to identify the cause of the performance issues on SalesSQLDb1. Which two dynamic management views should you use? Each correct answer presents part of the solution. NOTE: Each correct selection is worth one point.

**Options:**
A. sys.dm_pdw_nodes_tran_locks
B. sys.dm_exec_compute_node_errors
C. sys.dm_exec_requests
D. sys.dm_cdc_errors
E. sys.dm_pdw_nodes_os_wait_stats
F. sys.dm_tran_locks

**✅ Answer:** C, F

**Why:** Microsoft's Azure SQL Database blocking guide uses `sys.dm_exec_requests` (`blocking_session_id`, `wait_type`, `wait_resource`) to find blocked and head-blocker sessions. It uses `sys.dm_tran_locks` (joined to `sys.dm_os_waiting_tasks`) to see which locks are held and waited on. The `sys.dm_pdw_*` views (A, E) exist only in Azure Synapse dedicated SQL pools (PDW/SQL DW), not in Azure SQL Database. B is a PolyBase compute-node DMV and D is for change data capture; neither applies.

**Source-answer check:** DEMO said A, E ✘. Those are Synapse/PDW DMVs that don't exist in Azure SQL Database. The dump's own explanation uses sys.dm_tran_locks in its example query. The classic dump key differs from this answer.
**Confidence:** High (technically certain; note that the dump key A, E is common)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/understand-resolve-blocking

---

### Q156. Database watcher KQL elastic pool statements
<sub>ID U-169</sub>

**Sources:** MD-219   **Type:** Statements (Yes/No)   **Case study:** —

**Question:**
You have a logical server for Azure SQL Database named Server1 that has an elastic pool named Pool1. You have a database watcher named Watcher1. You run the following KQL query.
```kusto
let logicalServer = @"Server1";
let elasticPoolName = @"Pool1";
let replicaType = "Primary";
let duration = 20h;
sqldb_elastic_pool_resource_utilization
| where sample_time_utc > ago(duration)
| where logical_server_name =~ logicalServer
| where replica_type =~ replicaType
| project sample_time_utc,
  avg_cpu_percent,
  avg_instance_cpu_percent,
  avg_data_io_percent,
  avg_log_write_percent,
  max_worker_percent
| sort by sample_time_utc desc
| render timechart;
```
*(Source typos "logicelServer", "elacticPoolName" and "logical_server_nane" are corrected here.)*
For each of the following statements, select Yes if the statement is true. Otherwise, select No. NOTE: Each correct selection is worth one point.

**Options:**
- S1. The query returns the execution time of individual queries.
- S2. The query returns the resource consumption metrics from the last day.
- S3. The query visualizes the result set and includes a legend for each series.

**✅ Answer:** S1: **No**; S2: **No**; S3: **Yes**

**Why:** S1: `sqldb_elastic_pool_resource_utilization` holds pool-level resource samples (CPU, data IO, log write, workers), not per-query execution times. Those are in the query runtime statistics datasets. S2: `duration = 20h` with `ago(duration)` returns only the **last 20 hours**, not the last day (24 h). Microsoft's own example describes `duration = 1h` as "over the last one hour". The odd 20h value is the trap; a "Yes" statement would have used 1d or 24h. S3: `render timechart` draws a line chart with `sample_time_utc` on the x-axis and each numeric column as a series, with a legend shown by default.

**Source-answer check:** MD said No / Yes / Yes. S1 ✔, S2 ✘ (20 h ≠ last day), S3 ✔.
**Confidence:** Low for S2 (a loose reading, "every row is from within the last day", gives Yes), High for S1 and S3.   **Ref:** https://learn.microsoft.com/azure/azure-sql/database-watcher-analyze ; https://learn.microsoft.com/en-us/kusto/query/render-operator

---

### Q157. Monitor five SQL VMs in one view
<sub>ID U-170</sub>

**Sources:** MD-205   **Type:** MCQ   **Case study:** —

**Question:**
You have five instances of SQL Server on Azure Virtual Machines. You need to monitor Microsoft SQL Server performance for all the instances by consolidating metrics into a single graphic display. The solution must minimize administrative effort. What should you use?

**Options:**
A. Azure Monitor
B. Log Analytics
C. SQL Insights
D. Azure SQL Analytics

**✅ Answer:** A

**Why:** Azure Monitor is the built-in platform that collects metrics and performance counters from all five VMs and shows them together in one chart, dashboard or workbook (VM insights plus data collection rules). SQL Insights (preview), the option written for this scenario, was retired on 31 December 2024. Microsoft's replacement guidance for SQL Server on Azure VMs is VM Insights with DCRs in Azure Monitor. Azure SQL Analytics (D) is a legacy solution that covered only Azure SQL Database and Managed Instance, not SQL on VMs. Log Analytics (B) is just the workspace/query tool inside Azure Monitor, not a ready-made consolidated display.

**Source-answer check:** MD said A ✔ (current). Note: older dump keys/community answers for this item favored C (SQL Insights), which is now retired.
**Confidence:** Medium (the answer depends on the exam era; A is correct under current docs)   **Ref:** https://learn.microsoft.com/en-us/answers/a/1896644 ; https://learn.microsoft.com/en-us/azure/azure-sql/database/sql-insights-enable

---

### Q158. Diagnose tempdb contention on sqlsrv1 using DMVs
<sub>ID U-171</sub>

**Sources:** MD-187, NOTE-002-img081   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL Database server named sqlsrv1 that hosts 10 Azure SQL databases. The databases perform slower than expected. You need to identify whether the performance issue relates to the use of tempdb by Azure SQL databases on sqlsrv1. What should you do?

**Options:**
A. Run Query Store-based queries
B. Review information provided by SQL Server Profiler-based traces
C. Review information provided by Query Performance Insight
D. Run dynamic management view-based queries

**✅ Answer:** D

**Why:** Microsoft's "Monitor performance using DMVs" article has a section "Identify tempdb performance issues". It says to look for `PAGELATCH_*` waits (not `PAGEIOLATCH_*`) and to confirm in `sys.dm_exec_requests` that `wait_resource` starts with `2:x:y`, where database ID 2 is tempdb. Query Store and Query Performance Insight track query plans and runtime stats per database, not tempdb latch contention. SQL Server Profiler isn't supported for Azure SQL Database.

**Source-answer check:** All sources agree (D) ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/monitoring-with-dmvs

---

### Q159. Linux SQL VM performance baseline tooling
<sub>ID U-172</sub>

**Sources:** MD-217   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription that contains a SQL Server on Linux Azure Virtual Machines instance named VM1. VM1 contains a database named DB1. You need to create operational performance baselines for DB1 and VM1. What should you use?

**Options:**
A. InfluxDB, Collectd, and Grafana
B. Azure Monitor Log Analytics
C. Azure Event Hubs
D. top, grep, and sed

**✅ Answer:** A

**Why:** The DP-300 training unit "Establish baseline metrics" says: "If you're using Linux as the operating system, you need to install InfluxDB, Collectd, and Grafana to capture data similar to Windows Performance Monitor." The unit adds that these tools collect data from SQL Server and give you a graphical interface for correlating SQL Server and OS performance, which is what a baseline for both DB1 and VM1 needs. Log Analytics can collect VM metrics, but it doesn't capture SQL Server engine counters from a Linux VM by default, and it isn't the tool the official courseware names. Event Hubs only streams data. top, grep and sed are ad-hoc commands, not a baselining solution.

**Source-answer check:** MD said B ✘. The courseware names the InfluxDB/Collectd/Grafana stack for SQL Server on Linux, and the dump's reason for rejecting A has no support in Microsoft docs.
**Confidence:** Medium (directly backed by MS Learn training content; B is the most likely distractor and some dump keys use it)   **Ref:** https://learn.microsoft.com/en-us/training/modules/describe-performance-monitoring/4-establish-baseline-metrics

---

### Q160. Query to detect RESOURCE_SEMAPHORE waits
<sub>ID U-173</sub>

**Sources:** MD-183   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription that contains an Azure SQL database. The database fails to respond to queries in a timely manner. You need to identify whether the issue relates to resource_semaphore waits. How should you complete the Transact-SQL query?
```sql
SELECT [Box1],
   SUM(wait_time) AS total_wait_time_ms
FROM sys.[Box2] AS dmv1
  JOIN sys.dm_exec_sessions AS dmv2
    ON dmv1.session_id = dmv2.session_id
WHERE is_user_process = 1
GROUP BY wait_type
ORDER BY SUM(wait_time) DESC;
```

**Options:**
Box 1: is_user_process | wait_time | wait_type
Box 2: dm_exec_query_stats | dm_exec_requests | query_store_query

**✅ Answer:** Box 1: wait_type; Box 2: dm_exec_requests

**Why:** This matches the documented query in "Identify memory grant wait performance issues": `SELECT wait_type, SUM(wait_time) ... FROM sys.dm_exec_requests AS req INNER JOIN sys.dm_exec_sessions AS sess ... WHERE is_user_process = 1 GROUP BY wait_type`. The query groups by wait_type, so wait_type has to be the column you select. Only sys.dm_exec_requests has session_id, wait_type and wait_time. dm_exec_query_stats and query_store_query have no session_id or wait columns.

**Source-answer check:** MD ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/monitoring-with-dmvs

---

### Q161. Find database using most memory and I/O on SQLMI1
<sub>ID U-174</sub>

**Sources:** MD-201   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure SQL managed instance named SQLMI1 that hosts multiple databases. You need to monitor the performance of SQLMI1 and identify which database uses the most memory and the most disk I/O. Which objects should you query? To answer, select the appropriate options in the answer area.

**Options:**
Memory: sys.dm_exec_query_stats | sys.dm_io_virtual_file_stats | sys.dm_os_buffer_descriptors | sys.dm_os_performance_counters
Disk I/O: sys.dm_exec_query_stats | sys.dm_io_virtual_file_stats | sys.dm_os_buffer_descriptors | sys.dm_os_performance_counters

**✅ Answer:** Memory: sys.dm_os_buffer_descriptors; Disk I/O: sys.dm_io_virtual_file_stats

**Why:** sys.dm_os_buffer_descriptors returns one row per cached page and includes database_id. The documented example "Returning cached page count for each database" groups those pages by database, which shows which database uses the most buffer-pool memory. sys.dm_io_virtual_file_stats returns read/write counts, bytes and stalls per data and log file with database_id, so it shows which database generates the most disk I/O. sys.dm_os_performance_counters reports Memory Manager counters for the whole instance and can't break memory down by database. sys.dm_exec_query_stats is aggregated per cached plan, not per database.

**Source-answer check:** MD said Memory = sys.dm_os_performance_counters ✘ (those memory counters are instance-wide, not per database). Disk I/O = sys.dm_io_virtual_file_stats ✔.
**Confidence:** Medium (doc-backed reasoning; the dump key differs on the memory box)   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-os-buffer-descriptors-transact-sql

---

### Q162. SQLMI1 Intelligent Insights alerts at minimum cost
<sub>ID U-175</sub>

**Sources:** NOTE-93   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL managed instance named SQLMI1 that hosts 10 databases. You need to implement alerts by using Azure Monitor. The solution must meet the following requirements:
- Minimize costs.
- Aggregate Intelligent Insights telemetry from each database.

What should you do?

**Options:**
A. From the Diagnostic settings of each database, select Send to Log Analytics.
B. From the Diagnostic settings of each database, select Stream to an event hub.
C. From the Diagnostic settings of SQLMI1, select Send to Log Analytics.
D. From the Diagnostic settings of SQLMI1, select Stream to an event hub.

**✅ Answer:** A

**Why:** On SQL Managed Instance, the instance-level diagnostic setting has only the ResourceUsageStats category. SQLInsights (Intelligent Insights), Query Store statistics and Errors are categories of the **instance database** resource. Microsoft states that telemetry must be configured separately for the instance and for each instance database ("Repeat these steps for each instance database you want to monitor"). That rules out C and D, because SQLMI1's own settings can't send Intelligent Insights. A Log Analytics workspace aggregates all the databases' logs in one place and can drive Azure Monitor log alerts directly. An event hub is only a streaming pipe: it needs an extra consumer, costs more and doesn't raise alerts on its own.

**Source-answer check:** Note marked C ✘ (the instance-level diagnostic settings don't include the SQLInsights category).
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure

---

### Q163. Stream SQLMI1 metrics to Workspace1 first step
<sub>ID U-176</sub>

**Sources:** MD-197   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription that contains an Azure SQL managed instance named SQLMI1 and a Log Analytics workspace named Workspace1. You need to collect performance metrics for SQLMI1 and stream the metrics to Workspace1. What should you do first?

**Options:**
A. Create a private endpoint connection on SQLMI1.
B. Configure Azure SQL Analytics to use Workspace1.
C. Modify the Compute + storage settings for SQLMI1.
D. Modify the diagnostic settings for SQLMI1.

**✅ Answer:** D

**Why:** You stream managed-instance telemetry (ResourceUsageStats, plus metrics) by adding a diagnostic setting on the SQLMI1 resource and choosing "Send to Log Analytics workspace" → Workspace1. Azure SQL Analytics only reads data that's already in the workspace and is a retired/legacy solution, so it can't come first. A private endpoint and Compute + storage settings have nothing to do with telemetry export.

**Source-answer check:** MD ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure

---

### Q164. Free-space column in sys.database_files query
<sub>ID U-177</sub>

**Sources:** NOTE-8   **Type:** MCQ   **Case study:** —

**Question:**
You have the following Transact-SQL query.
```sql
SELECT
    [file_id] AS [File ID],
    [type] AS [File Type],
    substring([physical_name], 1,1) AS [Drive],
    [name] AS [Logical Name],
    [physical_name] AS [Physical Name],
    CAST([size] as DECIMAL(38,0))/128.0 AS [ColumnA],
    CAST(FILEPROPERTY([name], 'SpaceUsed') AS DECIMAL(38,0))/128.0 AS [ColumnB],
    (CAST([size] AS DECIMAL(38,0))/128.0) - (CAST(FILEPROPERTY([name], 'SpaceUsed') AS DECIMAL (38,0))/128.0) AS [ColumnC],
    [max_size] AS [ColumnD],
    [is_percent_growth] AS [Percent Growth Enabled],
    [growth] AS [Growth Rate],
    SYSDATETIME() AS [Current Date]
FROM sys.database_files;
```
Which column returned by the query represents the free space in each file?

**Options:**
A. ColumnA
B. ColumnB
C. ColumnC
D. ColumnD

**✅ Answer:** C

**Why:** `size` is the file size in 8-KB pages, and dividing by 128 converts it to MB, so ColumnA is the allocated size. `FILEPROPERTY(name,'SpaceUsed')` is the used pages, so ColumnB is the used MB. ColumnC = allocated − used = free space in MB. ColumnD (`max_size`) is the growth limit.

**Source-answer check:** Note ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/file-space-manage

---

### Q165. Retrieve db1 resource usage for last week
<sub>ID U-178</sub>

**Sources:** MD-156   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure SQL database named db1. You need to retrieve the resource usage of db1 from the last week. How should you complete the statement?
```sql
SELECT *
FROM [Box1]
WHERE database_name = 'db1' AND
      start_time > [Box2] (day, -7, GETDATE())
ORDER BY start_time DESC;
```

**Options:**
Box 1: sys.dm_db_resource_stats | sys.dm_exec_requests | sys.dm_user_db_resource_governance | sys.resource_stats
Box 2: DATEADD | DATEDIFF | DATEPART | TODATETIMEOFFSET

**✅ Answer:** Box 1: sys.resource_stats; Box 2: DATEADD

**Why:** sys.resource_stats lives in the logical server's master database. It has database_name and start_time columns, is aggregated every 5 minutes and keeps about 14 days, so it covers a week. sys.dm_db_resource_stats keeps only about 1 hour, at 15-second granularity, and has no database_name or start_time columns. `DATEADD(day, -7, GETDATE())` returns the point in time 7 days ago. DATEDIFF and DATEPART return integers, not a date to compare with.

**Source-answer check:** MD ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/monitoring-with-dmvs

---

### Q166. Read a Query Performance Insight top-5 CPU exhibit
<sub>ID U-179</sub>

**Sources:** MD-220   **Type:** Statements (Yes/No)   **Case study:** —

**Question:**
You have an Azure subscription that contains an Azure SQL database named DB1. You have the Query Performance Insight analysis shown in the following exhibit.

Exhibit: "TOP 5 queries by: CPU · Query aggregation: SUM · Time period: 08/02/2024 06:00:00 – 08/02/2024 10:00:00 · Metrics aggregation: AVG". Upper chart: a **red line, labelled in the legend "OVERALL DTU 1,34 %"**, rises to a peak of about **4%** (on a 0–4.5% axis). A dark-red bar for query 3964 reaches about **3.3–3.5%**. Lower chart: CPU (blue) 1,34 %, DATA IO 0 %, LOG IO 0,38 %.

| QUERY ID | CPU[%] | DATA IO[%] | LOG IO[%] | DURATION[hh:mm:ss] | EXECUTIONS COUNT |
|---|---|---|---|---|---|
| 3964 | 0.86 | 0 | 0 | 00:04:00.140 | 8 |
| 21195 | 0.03 | 0 | 0.02 | 00:00:01.180 | 12723 |
| 21198 | 0.02 | 0 | 0.01 | 00:00:00.850 | 9297 |
| 21196 | 0.01 | 0 | 0 | 00:00:00.250 | 2511 |
| 21197 | 0.01 | 0 | 0 | 00:00:00.630 | 19251 |

For each of the following statements, select Yes if the statement is true. Otherwise, select No.

**Options:**
S1. The peak DTU consumption is 4 percent.
S2. The peak consumption of the query that has ID 3964 is 4 percent.
S3. The query that has ID 21195 has a duration of 00:00:01.180 every time it runs.

**✅ Answer:** S1: Yes; S2: No; S3: No

**Why:** Microsoft's QPI documentation says: "The top line shows overall DTU percentage for the database. The bars show CPU percentage that the selected queries consumed." The red line, which the legend labels OVERALL DTU, peaks at about 4%, so S1 is true. Query 3964 is the dark-red bar, and it tops out at about 3.3–3.5%, not 4%, so S2 is false. Grid durations are aggregates that depend on the chosen aggregation function (Sum/Avg/Max) across 12,723 executions, not a per-execution constant, so S3 is false.

**Source-answer check:** MD said S1 No, S2 Yes ✘ (it read the red line as query 3964, but the legend and the docs identify that line as overall DTU). S3 No ✔.
**Confidence:** Medium (the exhibit is blurry. The answer rests on the doc-defined line-versus-bar meaning and on reading the bar height at about 3.4%.)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/query-performance-insight-use

---

### Q167. Keep Query Store from going read-only
<sub>ID U-180</sub>

**Sources:** MD-154, NOTE-27   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database named sqldb1. You need to minimize the possibility of Query Store transitioning to a read-only state. What should you do?

**Options:**
A. Double the value of Data Flush Interval
B. Halve (decrease by half) the value of Data Flush Interval
C. Double the value of Statistics Collection Interval
D. Halve (decrease by half) the value of Statistics Collection Interval

**✅ Answer:** B

**Why:** Per Microsoft, "Storage size is checked only when Query Store writes data to disk. This interval is set by the DATA_FLUSH_INTERVAL_SECONDS option…" and "If the Query Store has breached the MAX_STORAGE_SIZE_MB limit between storage size checks, it transitions to read-only mode." Flushing more often means size checks, and size-based cleanup, run more often. Doubling the interval does the opposite. Halving the Statistics Collection Interval (INTERVAL_LENGTH_MINUTES) creates more runtime-stats rows, which grows Query Store faster and makes the problem worse. Doubling it doesn't affect when size is checked.

**Source-answer check:** All sources agree (B) ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/performance/manage-the-query-store

---

### Q168. Make Query Store data available sooner for QPI
<sub>ID U-181</sub>

**Sources:** NOTE-87   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure SQL database named db1 on a server named server1. You use Query Performance Insight to monitor db1. You need to modify the Query Store configuration to ensure that performance monitoring data is available as soon as possible. Which configuration setting should you modify and which value should you configure?

**Options:**
Configuration setting: DATA_FLUSH_INTERVAL_SECONDS | INTERVAL_LENGTH_MINUTES | MAX_PLANS_PER_QUERY | QUERY_CAPTURE_MODE
Value: 1 | 60 | CUSTOM | ON

**✅ Answer:** Configuration setting: **INTERVAL_LENGTH_MINUTES**; Value: **1**

**Why:** Query Performance Insight shows Query Store runtime stats aggregated per collection interval. INTERVAL_LENGTH_MINUTES (allowed values 1, 5, 10, 15, 30, 60, 1440; default 60) sets that aggregation window. Microsoft says to "consider using a lower value if you require finer granularity or less time to detect and mitigate issues", and 1 minute is the minimum. DATA_FLUSH_INTERVAL_SECONDS only controls how often in-memory Query Store data is persisted to disk. The Query Store catalog views already merge in-memory data, so lowering it doesn't make data visible sooner. MAX_PLANS_PER_QUERY and QUERY_CAPTURE_MODE don't match the "available sooner" goal or the value list.

**Source-answer check:** The note marked DATA_FLUSH_INTERVAL_SECONDS (with 1) ✘. Many dumps give DATA_FLUSH_INTERVAL_SECONDS = 60, on the reading that "available" means "persisted sooner".
**Confidence:** Low (genuinely disputed. The documented behavior favors INTERVAL_LENGTH_MINUTES = 1, but if your exam key uses the flush interval, its pairing is DATA_FLUSH_INTERVAL_SECONDS = 60.)   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/performance/manage-the-query-store ; https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-set-options

---

### Q169. Force the fastest Query Store plan
<sub>ID U-182</sub>

**Sources:** NOTE-img025   **Type:** MCQ   **Case study:** —

**Question:**
*(Reconstructed: the stem above the exhibit wasn't captured and no full version was found online. It is presumably "You have an Azure SQL database… You review the Query Store Plan summary for a query…")*
Exhibit: Query Store "Plan summary for query 1178902". Y-axis = Total (duration), 0–250,000. Plan **1220917** (red) has two points at about 265,000 and about 220,000. Plan **1221065** (green) has one point near 0.
What should you do to ensure that the query uses the execution plan which executes in the least amount of time?

**Options:**
A. Force the query execution plan for plan 1221065.
B. Run the DBCC FREEPROCCACHE command.
C. Force the query execution plan for plan 1220917.
D. Disable parameter sniffing.

**✅ Answer:** A

**Why:** The plan summary shows that plan 1221065 has by far the lowest total duration. Forcing it in Query Store (Force Plan, or `sp_query_store_force_plan @query_id = 1178902, @plan_id = 1221065`) makes the optimizer always use that plan. Plan 1220917 is the slow plan. DBCC FREEPROCCACHE only causes a recompile, which might pick either plan again. Disabling parameter sniffing affects every query and doesn't guarantee the fast plan.

**Source-answer check:** Note marked A ✔
**Confidence:** Medium (the answer is clear from the exhibit, but the stem is partial and reconstructed)   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/performance/best-practice-with-the-query-store

---

### Q170. SIM: capture all queries in Query Store
<sub>ID U-183</sub>

**Sources:** MD-208   **Type:** Simulation (lab)   **Case study:** —

**Question:**
SIMULATION. You need to ensure that all queries executed against db1 are captured in the Query Store. To complete this task, sign in to the virtual machine. You may need to use SQL Server Management Studio and the Azure portal.

**Options:**
None (lab task).

**✅ Answer:** Query Store must be in Read write mode **and** Query Store Capture Mode must be **All**.
- **T-SQL** (SSMS or the portal Query editor, connected to db1):
  ```sql
  ALTER DATABASE [db1]
  SET QUERY_STORE = ON (OPERATION_MODE = READ_WRITE, QUERY_CAPTURE_MODE = ALL);
  ```
- **SSMS GUI:** right-click db1 → Properties → Query Store page → Operation Mode (Requested) = **Read write** → Query Store Capture Mode = **All** → OK.
- Optional check: `SELECT actual_state_desc, query_capture_mode_desc FROM sys.database_query_store_options;` should return READ_WRITE / ALL.

**Why:** In Azure SQL Database, Query Store is on by default but its capture mode is **AUTO**, which ignores infrequent and insignificant queries. Only `QUERY_CAPTURE_MODE = ALL` captures every query. Setting Read write alone, as the source's solution does, leaves AUTO in place, so it wouldn't capture all queries.

**Source-answer check:** MD said only "Operation Mode (Requested) = Read Write" ✘ (incomplete: it never sets Capture Mode = All, which is the actual requirement).
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/performance/manage-the-query-store ; https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-set-options

---

### Q171. SIM: set Query Store max size to 4000 MB
<sub>ID U-184</sub>

**Sources:** MD-224   **Type:** Simulation (lab)   **Case study:** —

**Question:**
SIMULATION. You need to increase the maximum storage size of the query store on db1 to 4,000 MB. You may need to use SQL Server Management Studio and the Azure portal.

**Options:**
None (lab task).

**✅ Answer:**
- **T-SQL** (SSMS or Azure portal → db1 → Query editor, connected to db1):
  ```sql
  ALTER DATABASE [db1] SET QUERY_STORE (MAX_STORAGE_SIZE_MB = 4000);
  ```
- **Or SSMS GUI:** right-click db1 → Properties → Query Store page → Max Size (MB) = **4000** → OK.
- Verify: `SELECT max_storage_size_mb FROM sys.database_query_store_options;` returns 4000.

**Why:** MAX_STORAGE_SIZE_MB sets the Query Store space quota. The Azure SQL Database maximum is 10,240 MB, so 4,000 is valid. The Azure portal has no GUI setting for this, so you use T-SQL or SSMS.

**Source-answer check:** MD ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/performance/manage-the-query-store

---

### Q172. Query Store actual read-only vs requested read-write
<sub>ID U-185</sub>

**Sources:** NOTE-011   **Type:** Hotspot   **Case study:** —

**Question:**
You have a database on a SQL Server on Azure Virtual Machines instance. The current state of Query Store for the database is shown in the following exhibit.

| Property | Value |
|---|---|
| Operation Mode (Actual) | Read only |
| Operation Mode (Requested) | Read write |
| Data Flush Interval (Minutes) | 15 |
| *(remaining rows, such as Max Size (MB), Query Store Capture Mode and Size Based Cleanup Mode, are cut off in the source screenshot)* | |

Use the drop-down menus to complete the statements.

**Options:**
Statement 1: To change Operation Mode (Actual) to Read write without losing any data, you must modify the [answer choice] setting: Max Size (MB) | Query Store Capture Mode | Size Based Cleanup Mode | Operation Mode (Requested)
Statement 2: Query Store will retain [answer choice] queries for evaluation: all | none of the | a selective set of

**✅ Answer:** Statement 1: **Max Size (MB)**; Statement 2: **a selective set of**

**Why:** The requested mode is already Read write, yet the actual mode is Read only. That is the classic sign that Query Store has used up MAX_STORAGE_SIZE_MB, and Microsoft documents that Query Store "changes its operation mode to READ_ONLY" when its maximum space is exhausted. Raising **Max Size (MB)** returns it to read-write without deleting anything. Turning on Size Based Cleanup Mode would free space by **purging** data, which breaks the "without losing any data" condition. Changing Operation Mode (Requested) does nothing because it is already Read write. For Statement 2, the hidden Capture Mode row is Auto in the standard version of this exhibit (the default since SQL Server 2019). Auto "ignores infrequent queries and queries with insignificant compile and execution duration", so only a selective set is retained.

**Source-answer check:** Note marked Statement 1 = Operation Mode (Requested) ✘ (it is already Read write. The quota must be raised.) Statement 2 = a selective set of ✔.
**Confidence:** Medium (Statement 1 is doc-backed. Statement 2 depends on the Capture Mode row, which the source screenshot cut off.)   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/performance/manage-the-query-store ; https://learn.microsoft.com/en-us/sql/relational-databases/performance/best-practice-with-the-query-store

---

### Q173. Resource Governor: App1 all CPU, half guaranteed
<sub>ID U-186</sub>

**Sources:** NOTE-136   **Type:** Drag-drop (sequence)   **Case study:** —

**Question:**
You have an Azure subscription that contains an Azure SQL managed instance, a database named db1, and an Azure web app named App1. App1 uses db1. You need to enable Resource Governor for App1. The solution must meet the following requirements:
- App1 must be able to consume all available CPU resources.
- App1 must have at least half of the available CPU resources always available.

Which three actions should you perform in sequence? (More than one order of answer choices is correct.)

**Options:**
Actions (visible list):
- Create a plan.
- Create a classifier function in db1.
- Create a workload group.
- Create a classifier function in the master database.
- Create a resource pool that has the following configurations: MAX_CPU_PERCENT = 100, MIN_CPU_PERCENT = 50

**✅ Answer:**
1. Create a resource pool that has MAX_CPU_PERCENT = 100, MIN_CPU_PERCENT = 50
2. Create a workload group (USING that pool)
3. Create a classifier function in the master database

**Why:** MIN_CPU_PERCENT = 50 reserves half the CPU for the pool under contention. MAX_CPU_PERCENT = 100 is a soft cap that lets the pool use all available CPU. A workload group is the container that sessions are classified into, and it must reference an existing pool. Microsoft states that the classifier function must be created "in the master database" with schema binding. A classifier in db1 is invalid, and "Create a plan" isn't a Resource Governor object. The classifier can be created before or after the workload group because it returns the group name as a string, which is why more than one order is accepted. The pool must come before the group.

**Source-answer check:** Note ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor-resource-pool ; https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor-classifier-function

---

### Q174. Limit App2 IOPS on managed instance SQL1
<sub>ID U-187</sub>

**Sources:** NOTE-182, MD-181   **Type:** Multi-select (choose two)   **Case study:** —

**Question:**
You have an Azure SQL managed instance named SQL1 and two Azure web apps named App1 and App2. You need to limit the number of IOPs that App2 queries generate on SQL1. Which two actions should you perform on SQL1? Each correct answer presents part of the solution.

**Options:**
A. Enable query optimizer fixes.
B. Enable Resource Governor.
C. Enable parameter sniffing.
D. Create a workload group.
E. Configure In-memory OLTP.
F. Run the Database Engine Tuning Advisor.
G. Reduce the Max Degree of Parallelism value.

**✅ Answer:** B, D

**Why:** Resource Governor is supported on SQL Managed Instance and can "reserve or limit the amount of CPU, memory, and physical I/O" that workloads use. Physical I/O is capped through the resource pool's MIN/MAX_IOPS_PER_VOLUME settings. You enable Resource Governor and create a workload group (bound to a pool with MAX_IOPS_PER_VOLUME) that App2's sessions are classified into. The other options are tuning knobs that don't put a cap on one app's I/O.

**Source-answer check:** NOTE said B, D ✔. MD gave no options or letters (its explanation points to Resource Governor + workload group, which is consistent).
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor ; https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor-resource-pool

---

### Q175. SQLMI1: limit App1 CPU and memory, four steps
<sub>ID U-188</sub>

**Sources:** MD-173, NOTE-90   **Type:** Drag-drop (sequence)   **Case study:** —

**Question:**
You have an Azure SQL managed instance named SQLMI1 that has Resource Governor enabled and is used by two apps named App1 and App2. You need to configure SQLMI1 to limit the CPU and memory resources that can be allocated to App1. Which four actions should you perform in sequence?

**Options:**
Actions:
- Create a workload group.
- Create a user-defined classifier function.
- Modify Resource Governor.
- Create a contained database user.
- Create a resource pool.

**✅ Answer:**
1. Create a resource pool.
2. Create a workload group.
3. Create a user-defined classifier function.
4. Modify Resource Governor.

**Why:** The resource pool holds the CPU/memory limits (MAX_CPU_PERCENT, MAX_MEMORY_PERCENT). The workload group is created USING that pool. The classifier UDF (in master, schema-bound) routes App1's sessions to the group. Finally, `ALTER RESOURCE GOVERNOR WITH (CLASSIFIER_FUNCTION = dbo.fn) ; ALTER RESOURCE GOVERNOR RECONFIGURE;` registers the classifier and applies the configuration, which Microsoft lists as required steps. A contained database user has nothing to do with resource governance.

**Source-answer check:** All sources agree ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor-classifier-function

---

### Q176. ADatum DB3 performance after migration
<sub>ID U-189</sub>

**Sources:** MD-147   **Type:** MCQ   **Case study:** ADatum v1 (financial services)

**Question:**
*(Case study: DB3 is on SVR3 (SQL Server 2019), is used by App2 and App3, and has its resource allocation managed by Resource Governor. It will be migrated to Azure SQL Database and added to an auto-failover group. Availability requirement: "For DB3, manage potential performance issues caused by resource demand changes by App2 and App3.")*
You need to recommend a solution to ensure that the performance of DB3 is optimized after the migration to Azure SQL Database. The solution must meet the availability requirements. What should you include in the recommendation?

**Options:**
A. vertical scaling
B. a custom resource pool
C. Resource Governor
D. horizontal scaling

**✅ Answer:** A

**Why:** In Azure SQL Database, "user configuration of resource pools and workload groups … isn't supported", so the on-premises Resource Governor approach (B and C) can't be carried over. Changing resource demand from App2/App3 is handled by scaling DB3's compute size or service tier up or down (vertical scaling). That is an online operation with only a brief reconnect, so it fits the availability requirement. Horizontal scaling (sharding or read scale-out) means redesigning the application and doesn't address shifting demand on a single database.

**Source-answer check:** MD ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor

---

### Q177. Data IO at 100% during a Table1 load
<sub>ID U-190</sub>

**Sources:** MD-193   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure SQL database named DB1 that contains a table named Table1. You run a query to load data into Table1. The performance metrics of Table1 during the load operation are shown in the following exhibit.

Exhibit: Azure portal metric chart, about 8:45 to after 9:05 (UTC-04:00). "Data IO percentage (Avg)" (blue): 0%, a brief spike to about 35%, then a **100% plateau** for most of the load, then a drop to a steady 25%. "Data IO percentage (Max)" (orange): 0% until the blue line drops, then 100% and stays there. Both legends show 100%.

Use the drop-down menus to select the answer choice that completes each statement based on the information presented in the graphic.

**Options:**
Statement 1: To reduce how long it takes to complete the query you must [answer choice]: scale the resource | use an elastic resource | perform query tuning
Statement 2: To reduce the log IO load of the operation, the query must be updated to use [answer choice] table: a temporary | an In-Memory OLTP durable | an In-Memory OLTP non durable

**✅ Answer:** Statement 1: **scale the resource**; Statement 2: **an In-Memory OLTP non durable** table

**Why:** Statement 1: Data IO % is measured against the service objective's limit and sits at 100% for most of the load, so the load is throttled by the tier's I/O cap. Microsoft's guidance: "consider increasing the performance level as these percentages near 100% on a consistent basis". "Use an elastic resource" (elastic pool) doesn't raise a single database's I/O cap. Statement 2: a memory-optimized table with `DURABILITY = SCHEMA_ONLY` (non-durable) generates no transaction-log I/O, which makes it ideal for staging loads. Durable memory-optimized tables still log, and #temp tables still log in tempdb.

**Source-answer check:** MD said "perform query tuning" ✘ (tuning is a generic remedy, but the exhibit shows the I/O cap being hit for a data load) + "In-Memory OLTP non durable" ✔.
**Confidence:** Low for Statement 1 (the docs list both "upgrade the compute size" and "tune queries" as remedies for hitting the I/O limit; some keys pick tuning), High for Statement 2.   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/monitoring-with-dmvs ; https://learn.microsoft.com/en-us/sql/relational-databases/in-memory-oltp/defining-durability-for-memory-optimized-objects

---

### Q178. Reduce Log IO percentage at minimum cost
<sub>ID U-191</sub>

**Sources:** MD-199   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database named DB1 in the General Purpose service tier. The performance metrics for DB1 are shown in the following exhibit.

*Exhibit:* "Log IO percentage (Max)" line chart, 12:25–12:52: 0% until about 12:42, then a flat 100% plateau until about 12:50, then a drop to about 37% and back to 0%.

You need to reduce the Log IO percentage. The solution must minimize costs. What should you do?

**Options:**
A. Change Service tier to Business Critical.
B. Increase the number of vCores.
C. Perform a checkpoint operation.
D. Change Recovery model to Simple.

**✅ Answer:** B

**Why:** In the General Purpose tier, the log rate limit scales with vCores (standard-series: 4.5 MiB/s per vCore, so 2 vCores = 9, 4 = 18, 8 = 36, up to a 50 MiB/s cap). Adding vCores therefore raises the log-rate ceiling and lowers Log IO %, and it costs less than Business Critical, which costs about 2.5–3x as much because it adds replicas. A checkpoint does not reduce log generation. Azure SQL Database always uses the FULL recovery model, so you cannot change it to Simple.

**Source-answer check:** MD said A ✘. Business Critical does have higher log rates, but it is not the minimum-cost fix. The MD explanation is about data-file IOPS, which is not what Log IO % measures. The classic dump key may still say A.
**Confidence:** Medium (no ExamTopics thread found to confirm which answer the exam expects. The docs support B as the cheapest effective fix.)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/resource-limits-vcore-single-databases

---

### Q179. SQL MI memory pressure, minimum cost
<sub>ID U-192</sub>

**Sources:** MD-204   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL managed instance named SQLMI1 that has the following settings:
- vCores: 4
- Service tier: General Purpose
- Hardware generation: Standard-series (Gen5)

You discover that memory pressure on SQLMI1 is high. You need to reduce the memory pressure on SQLMI1. The solution must minimize costs. What should you do?

**Options:**
A. Enable the Query Store.
B. Change vCores to 8.
C. Change Hardware generation to Premium-series.
D. Change Service tier to Business Critical.

**✅ Answer:** C

**Why:** Standard-series gives 5.1 GB per vCore (4 vCores = 20.4 GB). Premium-series gives 7 GB per vCore (4 vCores = 28 GB, about 37% more memory) for only about 15% more per vCore. Doubling to 8 vCores roughly doubles the compute and licence cost. Business Critical does not add memory per vCore (standard-series stays at 5.1 GB/vCore) and costs much more. Query Store does not reduce memory pressure.

**Source-answer check:** MD said B ✘. Doubling vCores does relieve memory pressure, but it is not the cheapest option now that premium-series hardware exists. Older dump keys (written before premium-series) say B.
**Confidence:** Medium (the answer depends on whether the exam treats premium-series as cheaper than doubling vCores. The docs and the Microsoft pricing blog show it is.)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/resource-limits

---

### Q180. Serverless transient errors after inactivity
<sub>ID U-193</sub>

**Sources:** MD-192, NOTE-101   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription that contains the resources shown in the following table.

| Name | Type |
|---|---|
| App1 | Azure web app |
| db1 | Azure SQL database in the serverless tier |

App1 experiences transient connection errors and timeouts when it attempts to access db1 after extended periods of inactivity. You need to modify db1 to resolve the issues experienced by App1 as soon as possible, without considering immediate costs. What should you do?

**Options (MD-192 order):**
A. Enable automatic tuning for db1.
B. Increase the number of vCores allocated to db1.
C. Decrease the auto-pause delay for db1.
D. Disable auto-pause delay for db1.

**✅ Answer:** D: Disable auto-pause delay for db1. (This is option B in the NOTE-101 option order.)

**Why:** A serverless database auto-pauses after the auto-pause delay. The first connection after a pause triggers an auto-resume, and that connection fails with a transient error until the database is online again. Disabling auto-pause keeps the database online all the time and costs more, which the question allows. A shorter delay makes pauses more frequent. More vCores and automatic tuning do not affect pause/resume.

**Source-answer check:** MD said D ✔. NOTE said B (same option, shuffled order) ✔. All sources agree.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview

---

### Q181. Elastic pool storage limit error 1132
<sub>ID U-194</sub>

**Sources:** CQ-043, MD-026   **Type:** MultiSelect   **Case study:** —

**Question:**
You have an Azure SQL Database elastic pool that contains 10 databases. You receive the following alert.

```
Msg 1132, Level 16, State 1, Line 1
The elastic pool has reached its storage limit. The storage used for the elastic pool cannot exceed (76800) MBs.
```

You need to resolve the alert. The solution must minimize administrative effort. Which three actions can you perform? Each correct answer presents a complete solution.

**Options:**
A. Increase the maximum storage of the elastic pool.
B. Delete data from a database.
C. Remove a database from the pool.
D. Enable data compression.
E. Shrink individual databases.

**✅ Answer:** A, C, E

**Why:** The pool limit applies to *allocated* data space. Each of these is a complete fix on its own: raise the pool max size, take a database (and its allocated space) out of the pool, or shrink data files to reclaim allocated-but-unused space. Deleting data (B) or compressing it (D) lowers *used* space but never lowers allocated space automatically, so neither fixes the error without a shrink afterwards.

**Source-answer check:** CQ said A, C, E ✔. MD-026 has no options or answer, so the options come from CQ.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/file-space-manage

---

### Q182. Minimize data and log file space
<sub>ID U-195</sub>

**Sources:** MD-186   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database named sqldb1. You need to minimize the amount of space used by the data and log files of sqldb1. What should you run?

**Options:**
A. DBCC SHRINKDATABASE
B. sp_clean_db_free_space
C. sp_clean_db_file_free_space
D. DBCC SHRINKFILE

**✅ Answer:** A

**Why:** `DBCC SHRINKDATABASE` shrinks all data **and** log files of the database in one command. `DBCC SHRINKFILE` targets one file per command. `sp_clean_db_free_space` and `sp_clean_db_file_free_space` only remove ghost-record residue from pages and do not reduce file size.

**Source-answer check:** MD said A ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/file-space-manage

---

### Q183. Tempdb PAGELATCH_UP: add tempdb files
<sub>ID U-197</sub>

**Sources:** MD-178   **Type:** YesNoSeries   **Case study:** —

**Question:**
*Note: This question is part of a series of questions that present the same scenario. Each question in the series contains a unique solution that might meet the stated goals.*

You have SQL Server 2019 on an Azure virtual machine. You are troubleshooting performance issues for a query in a SQL Server instance. To gather more information, you query `sys.dm_exec_requests` and discover that the wait type is `PAGELATCH_UP` and the wait_resource is `2:3:905856`. You need to improve system performance.

Solution: You create additional tempdb files.

Does this meet the goal?

**Options:**
A. Yes
B. No

**✅ Answer:** A (Yes)

**Why:** Wait resource `2:3:905856` means database 2 (tempdb), file 3, page 905856. 905856 = 8088 × 112, so this is a PFS page, and the waits show tempdb allocation-page (PFS) contention. Microsoft's fix is to add equally sized tempdb data files: one per logical processor up to 8, then more in steps of 4. Allocations are then spread across more PFS/GAM/SGAM pages.

**Source-answer check:** MD said Yes ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention

---

### Q184. Tempdb PAGELATCH_UP: master autogrow 10%
<sub>ID U-198</sub>

**Sources:** MD-175   **Type:** YesNoSeries   **Case study:** —

**Question:**
*(Same series scenario as Q183: SQL Server 2019 on an Azure VM. `sys.dm_exec_requests` shows wait type `PAGELATCH_UP` and wait_resource `2:3:905856`. You need to improve system performance.)*

Solution: You change the data file for the master database to autogrow by 10 percent.

Does this meet the goal?

**Options:**
A. Yes
B. No

**✅ Answer:** B (No)

**Why:** The contention is on tempdb (database_id 2) allocation pages, not on master (database_id 1). Changing master's autogrowth does nothing for latch contention in tempdb.

**Source-answer check:** MD said No ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention

---

### Q185. Tempdb PAGELATCH_UP: reduce temp objects
<sub>ID U-199</sub>

**Sources:** MD-176   **Type:** YesNoSeries   **Case study:** —

**Question:**
*(Same series scenario as Q183: SQL Server 2019 on an Azure VM. `sys.dm_exec_requests` shows wait type `PAGELATCH_UP` and wait_resource `2:3:905856`. You need to improve system performance.)*

Solution: You reduce the use of table variables and temporary tables.

Does this meet the goal?

**Options:**
A. Yes
B. No

**✅ Answer:** A (Yes)

**Why:** Tempdb allocation-page contention comes from creating and dropping many temp objects (temp tables and table variables) very often. Microsoft's guidance lists reducing that workload, or changing the code, as a mitigation alongside adding data files.

**Source-answer check:** MD said Yes ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention

---

### Q186. Tempdb PAGELATCH_UP: shrink transaction log
<sub>ID U-200</sub>

**Sources:** MD-174   **Type:** YesNoSeries   **Case study:** —

**Question:**
*(Same series scenario as Q183: SQL Server 2019 on an Azure VM. `sys.dm_exec_requests` shows wait type `PAGELATCH_UP` and wait_resource `2:3:905856`. You need to improve system performance.)*

Solution: You shrink the transaction log file.

Does this meet the goal?

**Options:**
A. Yes
B. No

**✅ Answer:** B (No)

**Why:** `PAGELATCH_UP` on a tempdb PFS page is in-memory latch contention on allocation pages in a data file. Shrinking a transaction log file does not touch that and can even hurt performance.

**Source-answer check:** MD said No ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention

---

### Q187. Tempdb data files after scaling to 16 vCPUs
<sub>ID U-201</sub>

**Sources:** MD-155   **Type:** MCQ   **Case study:** —

**Question:**
You have SQL Server 2019 on an Azure virtual machine that runs Windows Server 2019. The virtual machine has 4 vCPUs and 28 GB of memory. You scale up the virtual machine to 16 vCPUs and 64 GB of memory. You need to provide the lowest latency for tempdb. What is the total number of data files that tempdb should contain?

**Options:**
A. 2
B. 4
C. 8
D. 64

**✅ Answer:** C (8)

**Why:** With more than 8 logical processors, start with 8 equally sized tempdb data files. Add files in multiples of 4, up to the number of logical processors, only if contention continues. 64 files would be far too many, and 2 or 4 would leave contention.

**Source-answer check:** MD said C ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention

---

### Q188. Tempdb secondary data files at 8 vCPUs
<sub>ID U-202</sub>

**Sources:** MD-184   **Type:** MCQ   **Case study:** —

**Question:**
You have SQL Server 2019 on an Azure virtual machine that runs Windows Server 2019. The virtual machine has 4 vCPUs and 28 GB of memory. You scale up the virtual machine to 8 vCPUs and 64 GB of memory. You need to reduce tempdb contention without negatively affecting server performance. What is the number of secondary data files that you should configure for tempdb?

**Options:**
A. 2
B. 4
C. 8
D. 64

**✅ Answer:** C (8)

**Why:** With 8 or fewer logical processors, use one tempdb data file per logical processor, which is 8 here. Strictly speaking, 8 files means 1 primary plus 7 secondary files. 7 is not an option, and the exam treats "data files" and "secondary data files" as the same thing here, so 8 is the intended answer. 2 or 4 would leave contention, and 64 adds overhead with no benefit.

**Source-answer check:** MD said C ✔.
**Confidence:** Medium (the wording "secondary" is loose. 8 is the best available option.)   **Ref:** https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention

---

### Q189. Tempdb total data files at 8 vCPUs
<sub>ID U-203</sub>

**Sources:** MD-159, NOTE-047   **Type:** MCQ   **Case study:** —

**Question:**
You have SQL Server 2019 on an Azure virtual machine that runs Windows Server 2019. The virtual machine has 4 vCPUs and 28 GB of memory. You scale up the virtual machine to 8 vCPUs and 64 GB of memory. You need to provide the lowest latency for tempdb. What is the total number of data files that tempdb should contain?

**Options:**
A. 2
B. 4
C. 8
D. 64

**✅ Answer:** C (8)

**Why:** With 8 or fewer logical processors, use as many equally sized tempdb data files as logical processors, so 8 files for 8 vCPUs.

**Source-answer check:** MD said C ✔. NOTE said C ✔. All sources agree.
**Confidence:** High   **Ref:** https://learn.microsoft.com/troubleshoot/sql/performance/recommendations-reduce-allocation-contention

---


## D4 — Configure and manage automation of tasks (15–20%)

*28 questions*

### Q190. Email operator when Agent job fails
<sub>ID U-204</sub>

**Sources:** NOTE-177, NOTE-078   **Type:** DragDrop   **Case study:** —

**Question:**
You have SQL Server on an Azure virtual machine named SQL1. SQL1 has an agent job to back up all databases. You add a user named dbadmin1 as a SQL Server Agent operator. You need to ensure that dbadmin1 receives an email alert if a job fails.

Which three actions should you perform in sequence? To answer, move the appropriate actions from the list of actions to the answer area and arrange them in the correct order.

**Options:**
Actions:
- Create a job alert
- Create a job notification
- Enable Database Mail
- Enable the email settings for the SQL Server Agent
- Create a job target

**✅ Answer:**
1. Enable Database Mail
2. Enable the email settings for the SQL Server Agent
3. Create a job notification

**Why:** SQL Server Agent sends email through Database Mail, so Database Mail (with an account and profile) must exist first. Next, enable the mail profile on SQL Server Agent (Agent Properties > Alert System > Enable mail profile, then restart Agent). Last, on the job's Notifications page, set "Email dbadmin1 when the job fails". A job *alert* responds to error or performance events, not to a job's outcome. A job *target* is for multiserver (MSX/TSX) jobs.

**Source-answer check:** Both NOTE records (hand-numbered) give this order ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/database-mail/configure-sql-server-agent-mail-to-use-database-mail

---

### Q191. Agent job step access to file share
<sub>ID U-205</sub>

**Sources:** MD-043, CQ-060   **Type:** DragDrop   **Case study:** —

**Question:**
You have an Azure subscription that contains an instance of SQL Server on Azure Virtual Machines named SQLVM1 and a virtual machine named Server1 that runs Windows Server. SQLVM1 and Server1 are joined to an Active Directory Domain Services (AD DS) domain. Server1 hosts a file share named Share1.

You need to ensure that a SQL Server Agent job step on SQLVM1 can access the files in Share1. The solution must use the principle of least privilege.

Which three actions should you perform in sequence?

**Options:**
Actions:
- Assign the proxy to the job step.
- Create a login.
- Create a database user.
- Create a credential.
- Create a proxy.

**✅ Answer:**
1. Create a credential.
2. Create a proxy.
3. Assign the proxy to the job step.

**Why:** A proxy is built on a credential. The credential stores a domain account that has rights on Share1. Once the proxy exists, you enable it for the right subsystem (for example CmdExec or PowerShell) and select it in the job step's "Run as". Only that job step then runs under the share-access account, not the whole Agent service account, which is least privilege. A login or database user does not give access to the file system.

**Source-answer check:** MD ✔ and CQ ✔. All sources agree.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/ssms/agent/create-a-sql-server-agent-proxy

---

### Q192. Enable Agent email notifications on SQL MI
<sub>ID U-206</sub>

**Sources:** NOTE-068   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL managed instance. You need to enable SQL Agent Job email notifications. What should you do?

**Options:**
A. Use the Agent XPs option.
B. Enable the SQL Server Agent.
C. Run the sp_configure command.
D. Run the sp_set_agent_properties command.

**✅ Answer:** C

**Why:** On SQL Managed Instance you enable Database Mail with `sp_configure` (`'show advanced options', 1`, then `'Database Mail XPs', 1`, then `RECONFIGURE`). You then create a Database Mail profile named `AzureManagedInstance_dbmail_profile` and assign operators and notifications to the jobs. Microsoft Learn says plainly that SQL Agent settings are read-only on MI and that `sp_set_agent_properties` isn't supported, so D cannot work. SQL Agent always runs on MI and cannot be enabled or disabled (B). Agent XPs (A) has nothing to do with mail.

**Source-answer check:** NOTE said D ✘. `sp_set_agent_properties` is explicitly unsupported on SQL Managed Instance. Older dump keys may show D.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/job-automation-managed-instance ; https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/transact-sql-tsql-differences-sql-server

---

### Q193. MSX/TSX multiserver administration deployment choice
<sub>ID U-207</sub>

**Sources:** MD-028, CQ-045   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription. You plan to migrate 10 on-premises Microsoft SQL Server instances to Azure. You need to ensure that the migrated environment can be managed by using multiserver administration and supports master/target (MSX/TSX) jobs. The solution must minimize administrative effort. Which SQL deployment options should you select as the master server (MSX) and the target server (TSX)?

**Options:**
- MSX: SQL database | SQL managed instances | SQL virtual machines
- TSX: SQL database | SQL managed instances | SQL virtual machines

**✅ Answer:** MSX: **SQL virtual machines**. TSX: **SQL virtual machines**.

**Why:** Azure SQL Database has no SQL Server Agent at all. Microsoft Learn's MI T-SQL differences page says: "The Multi Server Administration feature for master/target (MSX/TSX) jobs aren't supported." Only full SQL Server (on Azure VMs) supports both MSX and TSX roles. Both the master and the targets must therefore be SQL Server on Azure VMs.

**Source-answer check:** MD and CQ both said MSX = MI and TSX = MI ✘. The dump explanation relies on the "Applies to" header of the "Make a Master Server" page, but the MI limitations page says MSX/TSX is not supported. The classic dump key (MI/MI) predates or ignores this limitation.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/transact-sql-tsql-differences-sql-server

---

### Q194. OS right for SQL Agent account
<sub>ID U-208</sub>

**Sources:** MD-112   **Type:** MCQ   **Case study:** —

**Question:**
You have an instance of SQL Server on Azure Virtual Machines named VM1. You plan to schedule a SQL Server Agent job that will rebuild indexes of the databases hosted on VM1. You need to configure the account that will be used by the agent. The solution must use the principle of least privilege. Which operating system user right should you assign to the account?

**Options:**
A. Increase scheduling priority
B. Log on as a service
C. Profile system performance
D. Log on as a batch job

**✅ Answer:** B

**Why:** The SQL Server Agent service account needs the Windows right **Log on as a service** (SeServiceLogonRight), together with a few default token and quota rights. An index rebuild runs inside the database engine, so no other OS rights are needed. "Log on as a batch job" applies to scheduled-task and proxy contexts, and the other two rights are unrelated.

**Source-answer check:** MD said B ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/previous-versions/sql/2014/ssms/agent/configure-sql-server-agent?view=sql-server-2014

---

### Q195. sp_add_alert severity 20 notification statements
<sub>ID U-209</sub>

**Sources:** MD-218   **Type:** Statements(Yes/No)   **Case study:** —

**Question:**
You have an Azure subscription that contains a SQL Server instance on a virtual machine named Server1. You execute the following T-SQL statement on Server1.

```sql
EXEC msdb.dbo.sp_add_alert @name=N'Fatal Error',
  @message_id=0,
  @severity=20,
  @enabled=1,
  @delay_between_responses=0,
  @include_event_description_in=1;
GO

EXEC msdb.dbo.sp_add_operator
  @name = 'DBA-Team',
  @enabled = 1,
  @email_address = 'DBA-Team@contoso.com';
GO

EXEC msdb.dbo.sp_add_notification
  @alert_name = N'Fatal Error',
  @operator_name = 'DBA-Team',
  @notification_method = 1;
GO
```

For each of the following statements, select Yes if the statement is true, otherwise select No.

**Options:**
- S1. An alert will be generated when a fatal error occurs.
- S2. A SQL Server Agent job will run when an alert is generated.
- S3. An email will be sent to DBA-Team@contoso.com when an alert is generated.

**✅ Answer:** S1 **Yes**. S2 **No**. S3 **Yes**.

**Why:** Severity 20 is a fatal error ("fatal error in current process"). Errors of severity 19–25 are always written to the Windows application log, so the enabled alert fires (S1). `sp_add_alert` gives no `@job_id`/`@job_name`, so no job runs in response (S2). `sp_add_notification` with `@notification_method = 1` means email to the DBA-Team operator (S3). This assumes Agent mail (Database Mail) is configured, which is the exam's intended reading.

**Source-answer check:** MD said Y/N/Y ✔.
**Confidence:** Medium (the answer for S3 assumes Database Mail and Agent mail are configured, which the question doesn't show.)   **Ref:** https://learn.microsoft.com/ro-ro/ssms/agent/create-an-alert-using-severity-level

---

### Q196. Role to configure Agent proxy accounts
<sub>ID U-210</sub>

**Sources:** MD-118   **Type:** MCQ   **Case study:** —

**Question:**
You have an instance of SQL Server on Azure Virtual Machines. You need to ensure that a user named User1 can configure proxy accounts for SQL Server Agent jobs. The solution must use the principle of least privilege. Which role should you assign to User1?

**Options:**
A. sysadmin
B. SQLAgentUserRole
C. SQLAgentReaderRole
D. SQLAgentOperatorRole

**✅ Answer:** A

**Why:** Only members of the **sysadmin** fixed server role can create, modify, or delete SQL Server Agent proxy accounts. The msdb Agent roles can at most *use* proxies granted to them, and SQLAgentOperatorRole can view proxy properties. None of them can create proxies, even though they look less privileged.

**Source-answer check:** MD said A ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/ssms/agent/create-a-sql-server-agent-proxy ; https://learn.microsoft.com/en-us/ssms/agent/sql-server-agent-fixed-database-roles

---

### Q197. Agent role to create and view jobs
<sub>ID U-211</sub>

**Sources:** MD-196   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription that contains an instance of SQL Server on Azure Virtual Machines named SQLVM1 and a user named User1. SQLVM1 hosts a database named DB1. You need to ensure that User1 can perform the following tasks on DB1:
- Create jobs.
- View all jobs.
- Modify, delete, and disable the jobs the user created.

The solution must use the principle of least privilege. Which built-in database role should you assign to User1, and where is the role defined?

**Options:**
- Location: DB1 | master | msdb
- Built-in role: SQLAgentOperatorRole | SQLAgentReaderRole | SQLAgentUserRole

**✅ Answer:** Location: **msdb**. Built-in role: **SQLAgentReaderRole**.

**Why:** The SQL Server Agent fixed database roles exist only in **msdb**. SQLAgentUserRole lets a user create and manage only their *own* jobs and see only those, so it can't "view all jobs". SQLAgentReaderRole adds viewing of all local and multiserver jobs and their history, and still limits modify, delete, and enable/disable to owned jobs. That is an exact match. SQLAgentOperatorRole also lets the user execute, stop, and enable/disable *all* jobs, which is more than needed.

**Source-answer check:** MD said msdb / SQLAgentReaderRole ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/ssms/agent/sql-server-agent-fixed-database-roles

---

### Q198. Least-privilege Agent role to schedule backup job
<sub>ID U-212</sub>

**Sources:** MD-113   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription that contains an instance of SQL Server on an Azure virtual machine named SQLVM1 and a user named User1. SQLVM1 hosts a database named DB1. User1 has the db_backupoperator role. You need to ensure that User1 can create a scheduled task to perform a full backup of DB1. The solution must use the principle of least privilege. Which built-in database role should you assign to User1?

**Options:**
A. db_owner
B. SQLAgentReaderRole
C. SQLAgentUserRole
D. SQLAgentOperatorRole

**✅ Answer:** C

**Why:** A scheduled task on SQL Server = a SQL Server Agent job. SQLAgentUserRole (msdb) is the least-privileged Agent role and can create/modify/delete and run its own local jobs and job schedules. SQLAgentReaderRole and SQLAgentOperatorRole include those rights plus more (so not least privilege); db_owner on DB1 grants no Agent access and is far broader. User1 already has db_backupoperator, so the backup itself is permitted.

**Source-answer check:** MD said C ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/ssms/agent/sql-server-agent-fixed-database-roles

---

### Q199. Role needed to create Agent proxy
<sub>ID U-213</sub>

**Sources:** MD-124   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription that contains an instance of SQL Server on an Azure virtual machine named VM1 and a Microsoft Entra Domain Services domain that contains two users named User1 and User2. On the default instance of SQL Server on VM1, you create a credential named Credential1 for User1. You need to ensure that User2 can create a SQL Server Agent proxy that will use Credential1. The solution must use the principle of least privilege. Which role should you assign to User2?

**Options:**
A. SQLAgentUserRole
B. SQLAgentReaderRole
C. SQLAgentOperatorRole
D. sysadmin

**✅ Answer:** D

**Why:** Only members of the sysadmin fixed server role can create, modify, or delete SQL Server Agent proxy accounts. The three msdb Agent roles can only *use* proxies they were granted (Create/modify/delete Proxies = No even for SQLAgentOperatorRole), so sysadmin is the least privilege that works.

**Source-answer check:** MD said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/ssms/agent/create-a-sql-server-agent-proxy ; https://learn.microsoft.com/en-us/ssms/agent/sql-server-agent-fixed-database-roles

---

### Q200. Alert resource type for CPU on all databases
<sub>ID U-214</sub>

**Sources:** MD-158, NOTE-036, NOTE-62   **Type:** MCQ   **Case study:** —

**Question:**
You have several Azure SQL databases on the same Azure SQL Database server in a resource group named ResourceGroup1. You must be alerted when CPU usage exceeds 80 percent for any database. The solution must apply to any additional databases that are created on the Azure SQL server. Which resource type should you use to create the alert?

**Options:**
A. Resource Groups
B. SQL Servers
C. SQL Databases
D. SQL Virtual Machines

**✅ Answer:** C

**Why:** CPU percentage is a metric of the database resource (Microsoft.Sql/servers/databases), and Azure Monitor supports *multi-resource* metric alert rules for "SQL Server databases" (Microsoft.Sql/servers/databases). In the portal you pick the resource type SQL databases and scope the rule to the server/resource group (not individual DBs), so one rule covers every current and future database of that type in that scope. The logical server (Microsoft.Sql/servers) resource type does not expose the per-database CPU percent metric for a multi-resource rule; "Resource Groups" is only the scope container, not the resource type the metric belongs to; SQL VMs are irrelevant.

**Source-answer check:** MD said C ✔; NOTE-036 and NOTE-62 marked B (SQL Servers) ✘ — the server is the scope you choose, but the alert's resource type (where the CPU metric lives and multi-resource alerting is supported) is SQL databases. The commonly published key is C.
**Confidence:** Medium   **Ref:** https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-types ; https://learn.microsoft.com/en-us/azure/azure-sql/database/alerts-create?view=azuresql ("configure alerts for all resources of a type in a subscription, resource group, or an Azure SQL Database logical server")

---

### Q201. Notify on database size/tier changes
<sub>ID U-215</sub>

**Sources:** MD-157, NOTE-080   **Type:** MCQ   **Case study:** —

**Question:**
You have 50 Azure SQL databases. You need to notify the database owner when the database settings, such as the database size and pricing tier, are modified in Azure. What should you do?

**Options:**
A. Create a diagnostic setting for the activity log that has the Security log enabled.
B. For the database, create a diagnostic setting that has the InstanceAndAppAdvanced metric enabled.
C. Create an alert rule that uses a Metric signal type.
D. Create an alert rule that uses an Activity Log signal type.

**✅ Answer:** D

**Why:** Changing size or pricing tier is a control-plane write operation (Microsoft.Sql/servers/databases/write) recorded in the Azure activity log; an activity log alert rule fires on that event and notifies via an action group. Diagnostic settings (A, B) only route logs/metrics and send no notification; a metric alert (C) watches numeric values, not configuration change events.

**Source-answer check:** MD & NOTE said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-types

---

### Q202. SIM: email alert db1 CPU over 50%
<sub>ID U-216</sub>

**Sources:** MD-222   **Type:** Simulation   **Case study:** —

**Question:**
SIMULATION. You need to generate an email alert for db1 if the average CPU percentage utilization is greater than 50 percent for five minutes sampled at one-minute intervals. The alert must be sent to admin@contoso.com. You may need to use SQL Server Management Studio and the Azure portal.

**Options:**
None (lab task).

**✅ Answer:** Create an Azure Monitor metric alert rule on db1 (Azure portal):
1. Azure portal → SQL databases → **db1** → **Monitoring > Alerts** → **+ Create > Alert rule** (scope = db1).
2. **Condition** tab → See all signals → **CPU percentage** → Apply.
3. Threshold = **Static**, Aggregation type = **Average**, Operator = **Greater than**, Threshold value = **50**.
4. When to evaluate: **Check every = 1 minute**, **Lookback period = 5 minutes** (older UI: Frequency of evaluation 1 minute, Aggregation granularity/period 5 minutes).
5. **Actions** tab → **Create action group** (or quick action) → Notifications: type **Email/SMS message/Push/Voice** → Email = **admin@contoso.com** → OK → create/attach the action group.
6. **Details** tab → enter an alert rule name (e.g., db1-cpu-50), severity as desired, Enable upon creation.
7. **Review + create** → **Create**.

**Why:** CPU percentage is a platform metric of the database, so a metric alert with Average aggregation over a 5-minute window evaluated every minute matches "average > 50% for five minutes sampled at one-minute intervals"; email delivery is done by an action group. SSMS is not needed.

**Source-answer check:** MD's step list matches ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/alerts-create?view=azuresql

---

### Q203. Reduce noisy alerts: dynamic threshold, low sensitivity
<sub>ID U-217</sub>

**Sources:** MD-185   **Type:** MultiSelect   **Case study:** —

**Question:**
You receive numerous alerts from Azure Monitor for an Azure SQL Database instance. You need to reduce the number of alerts. You must only receive alerts if there is a significant change in usage patterns for an extended period. Which two actions should you perform? Each correct answer presents part of the solution. NOTE: Each correct selection is worth one point.

**Options:** (missing in MD-185; recovered from the public pass4success/ExamTopics copy of the question)
A. Set Threshold Sensitivity to High
B. Set the Alert logic threshold to Dynamic
C. Set the Alert logic threshold to Static
D. Set Threshold Sensitivity to Low
E. Set Force Plan to On

**✅ Answer:** B, D

**Why:** Dynamic thresholds learn the metric's historical/seasonal pattern and alert on significant deviations from it. With threshold sensitivity Low, thresholds are loose and alerts fire only on large deviations, giving the fewest alerts (High = tight thresholds = more alerts). Static thresholds don't model usage patterns; Force Plan is an automatic-tuning setting unrelated to alerting.

**Source-answer check:** MD said B, D ✔ (option letters per recovered list)
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-dynamic-thresholds ; https://learn.microsoft.com/en-us/azure/azure-sql/database/alerts-create?view=azuresql

---

### Q204. Voice notification for MI maintenance events
<sub>ID U-218</sub>

**Sources:** NOTE-037   **Type:** MCQ   **Case study:** —

**Question:**
You manage 100 Azure SQL managed instances located across 10 Azure regions. You need to receive voice message notifications when a maintenance event affects any of the 10 regions. The solution must minimize administrative effort. What should you do?

**Options:**
A. From the Azure portal, create a service health alert.
B. From the Azure portal, create an Azure Advisor operational excellence alert.
C. From Microsoft SQL Server Management Studio (SSMS), configure a SQL Server agent job.
D. From the Azure portal, configure an activity log alert.

**✅ Answer:** A

**Why:** Service Health alerts notify you of outages and planned maintenance affecting the services/regions you use; one rule can cover all 10 regions and SQL Managed Instance, and its action group can include a Voice notification. Advisor alerts are for recommendations; an Agent job can't see Azure maintenance; a generic activity log alert on resource operations isn't the purpose-built (Service Health category) option — Service Health alerts are technically a type of activity log alert, but A is the intended, least-effort answer.

**Source-answer check:** NOTE said A ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-types ; https://learn.microsoft.com/en-us/azure/azure-sql/database/alerts-create?view=azuresql

---

### Q205. Litware: SalesSQLDb1 statistics maintenance runbook
<sub>ID U-219</sub>

**Sources:** NOTE-img059   **Type:** DragDrop   **Case study:** Litware

**Question:**
(Case study — Litware. SalesSQLDb1 is an Azure SQL database in elastic pool SalesSQLDb1Pool and has performance issues likely due to out-of-date statistics. Technical requirements: maintenance tasks must be automated; use of on-premises infrastructure must be minimized.)
You need to implement statistics maintenance for SalesSQLDb1. The solution must meet the technical requirements. Which four actions should you perform in sequence? To answer, move the appropriate actions from the list of actions to the answer area and arrange them in the correct order.
*(Source screenshot is missing the question header; stem and actions are complete.)*

**Options:**
Actions:
- Create and configure a schedule.
- Create a SQL Server Agent job.
- Publish the runbook.
- Create an Azure Automation account.
- Import the SqlServer module.
- Create a runbook that runs a PowerShell script.
- Run sp_add_jobserver.

**✅ Answer:** 1. Create an Azure Automation account → 2. Import the SqlServer module → 3. Create a runbook that runs a PowerShell script → 4. Publish the runbook

**Why:** Azure SQL Database (single DB/elastic pool) has no SQL Server Agent, so the Agent-job and sp_add_jobserver options are out, and running them from an on-premises or VM SQL Server would violate "minimize on-premises infrastructure". Azure Automation is the cloud option. Create the account, then import the SqlServer module (it provides Invoke-Sqlcmd and isn't there by default). Next, write the PowerShell runbook that updates statistics, then publish it, because only a published runbook can be started or linked to a schedule. In practice you'd then link a schedule as a fifth step; Microsoft's DP-300 lab does account → SqlServer module → credential → runbook → publish → schedule. With only four slots, publish must come first, so the schedule is the step left out.

**Source-answer check:** NOTE (hand-numbered) said account → module → runbook → publish ✔
**Confidence:** Medium (some candidates argue for replacing "Import the SqlServer module" with "Create and configure a schedule"; the commonly accepted key is the one above)   **Ref:** https://microsoftlearning.github.io/dp-300-database-administrator/Instructions/Labs/13-deploy-automation-runbook-rebuild-indexes.html

---

### Q206. Recreate App1Dev in App1Test (ARM + SSDT)
<sub>ID U-220</sub>

**Sources:** MD-005, CQ-007   **Type:** DragDrop   **Case study:** —

**Question:**
You have a resource group named App1Dev that contains an Azure SQL Database server named DevServer1. DevServer1 contains an Azure SQL database named DB1. The schema and permissions for DB1 are saved in a Microsoft SQL Server Data Tools (SSDT) database project. You need to populate a new resource group named App1Test with the DB1 database and an Azure SQL Server named TestServer1. The resources in App1Test must have the same configurations as the resources in App1Dev. Which four actions should you perform in sequence?

**Options:**
Actions:
- Change the Active Directory Admin on TestServer1
- Change the server name and related variables in the templates
- From the database project, deploy the database schema and permissions
- Add IP addresses to the firewall
- From the Azure portal, export the Azure Resource Manager templates
- From the Azure portal, deploy the templates.

**✅ Answer:** 1. From the Azure portal, export the Azure Resource Manager templates → 2. Change the server name and related variables in the templates → 3. From the Azure portal, deploy the templates → 4. From the database project, deploy the database schema and permissions

**Why:** Exporting the App1Dev ARM templates captures the existing server/database configuration (including firewall rules and Entra admin), so nothing needs to be changed by hand. Edit the server name (TestServer1) and related variables, deploy to App1Test, then publish the SSDT project to recreate DB1's schema and permissions. Changing the AD admin would make the config differ from App1Dev, and the firewall IPs are already in the template.

**Source-answer check:** MD & CQ agree ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/training/modules/configure-automatic-deployment-azure-sql-database/3-automate-deployment-using-azure-resource-manager-bicep

---

### Q207. ARM template: server type and dependsOn
<sub>ID U-221</sub>

**Sources:** MD-017, CQ-034   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription. You plan to deploy an Azure SQL database by using an Azure Resource Manager template. How should you complete the template?
```
{
  "resources": [
   {
    "type": [Box 1],
    "apiVersion": "2020-02-02-preview",
    "name": "[parameters('name1')]",
    "location": "[parameters('location')]",
   ...
    "resources": [
      {
       "type": "databases",
       "apiVersion": "2020-02-02-preview",
   ...
       [Box 2]
       "[resourceId('Microsoft.Sql/servers',concat(parameters('name1')))]"
   ...
  }
```

**Options:**
Box 1: "Microsoft.Sql/servers" | "Microsoft.SqlVirtualMachines/sqlVirtualMachines" | "Microsoft.Synapse/workspaces/sqldatabases"
Box 2: "dependsOn":[ | "properties":[ | "tags":[

**✅ Answer:** Box 1: "Microsoft.Sql/servers"; Box 2: "dependsOn":[

**Why:** An Azure SQL database is a child (databases) of a logical server, resource type Microsoft.Sql/servers. The child database must be deployed after the server, expressed by a dependsOn array containing resourceId('Microsoft.Sql/servers', name). properties and tags are objects, not lists of resource IDs.

**Source-answer check:** MD & CQ agree ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/sql-database/sql-database-single-database-get-started-template

---

### Q208. ARM template: MI for cross-database queries
<sub>ID U-222</sub>

**Sources:** MD-014, CQ-031   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription. You need to deploy an Azure SQL resource that will support cross database queries by using an Azure Resource Manager (ARM) template. How should you complete the ARM template?
```
"resources": [
  ...
  "type": [Box 1],
  "name": "[parameters('targetName')]",
  "location": "[parameters('location')]",
  "sku": { "name": "[parameters('skuName')]" ... },
  "dependsOn": [ [Box 2] ],
  "properties": {
    "administratorLogin": "[parameters('administratorLogin')]",
    "administratorLoginPassword": "[parameters('administratorLoginPassword')]",
    "subnetId": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworkName'), parameters('subnetName'))]",
    "storageSizeInGB": "[parameters('storageSizeInGB')]",
    "vCores": "[parameters('vCores')]",
    "licenseType": "[parameters('licenseType')]"
  ...
]
```

**Options:**
Box 1: Microsoft.Sql/servers | Microsoft.Sql/servers/databases | Microsoft.Sql/managedInstances
Box 2: "[parameters('targetName')]" | "[parameters('virtualNetworkName')]" | "[variables('networkSecurityGroupName')]"

**✅ Answer:** Box 1: Microsoft.Sql/managedInstances; Box 2: "[parameters('virtualNetworkName')]"

**Why:** Native cross-database (three-part name) queries are supported in SQL Managed Instance, not Azure SQL Database. The template's subnetId, vCores, storageSizeInGB and licenseType properties are also managed-instance properties. A managed instance is deployed into a VNet subnet, so it depends on the virtual network. The NSG is a dependency of the subnet, not of the instance, and targetName is the instance itself.

**Source-answer check:** MD & CQ agree ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-only-authentication-create-server?view=azuresql (MI ARM template: `"dependsOn": ["[parameters('virtualNetworkName')]"]`) ; https://learn.microsoft.com/ko-kr/azure/azure-sql/managed-instance/create-template-quickstart?view=azuresql-mi

---

### Q209. ARM template statements: Standard capacity 10
<sub>ID U-223</sub>

**Sources:** MD-015, CQ-032   **Type:** Statements(Yes/No)   **Case study:** —

**Question:**
You have the following Azure Resource Manager template.
```
"variable": { "serverName": "azsqldbserver0001" },
"resources": [
  {
    "name": "[variables('serverName')]",
    "type": "Microsoft.Sql/servers",
    "apiVersion": "2019-06-01-preview",
    "location": "[parameters('location')]",
    "properties": {
      "administratorLogin": "[parameters('administratorLogin')]",
      "administratorLoginPassword": "[parameters('administratorLoginPassword')]",
      "version": "12.0"
    },
    "resources": [
      {
        "name": "[concat(variables('serverName'),'/',parameters('databaseName'))]",
        "type": "Microsoft.Sql/servers/databases",
        "apiVersion": "2020-08-01-preview",
        "location": "[parameters('location')]",
        "kind": "v12.0",
        "sku": { "name": "Standard", "tier": "Standard", "capacity": 10 },
        "dependsOn": [ "[concat('Microsoft.Sql/servers/', variables('serverName'))]" ],
        "properties": { },
        "resources": [ ]
      }
    ]
  }
],
```
For each of the following statements, select Yes if the statement is true. Otherwise, select No.

**Options:**
1. The template deploys a serverless Azure SQL database.
2. The template deploys a database to an Azure SQL Database managed instance.
3. The pricing tier of the database deployment is based on DTUs.

**✅ Answer:** 1: No; 2: No; 3: Yes

**Why:** sku Standard/Standard with capacity 10 = S0 (10 DTUs) in the DTU purchasing model. Serverless exists only in the vCore model (GP_S_Gen5 / Hyperscale serverless SKUs). The resource types are Microsoft.Sql/servers and servers/databases (logical server + single database), not Microsoft.Sql/managedInstances.

**Source-answer check:** MD & CQ agree ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/sql-database/sql-database-single-database-get-started-template

---

### Q210. az sql db create required parameters
<sub>ID U-224</sub>

**Sources:** MD-027, CQ-044   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription. You need to deploy a new Azure SQL database by using Azure Command-Line Interface (CLI). Which three parameters are required?

**Options:**
A. --name, --edition, and --capacity
B. --name, --tier, and --min-capacity
C. --name, --resource-group, and --server
D. --name, --licence-type, and --capacity

**✅ Answer:** C

**Why:** The required parameters of `az sql db create` are `--name/-n`, `--resource-group/-g` and `--server/-s`. Edition, capacity, tier, min-capacity and license type are optional; defaults apply (or they're needed only for vCore/serverless configurations).

**Source-answer check:** MD & CQ said C ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/cli/azure/sql/db?view=azure-cli-latest

---

### Q211. Deploy database from BACPAC with CLI
<sub>ID U-225</sub>

**Sources:** MD-071   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription. You need to deploy an Azure SQL database by using a BACPAC file. Which command should you run?

**Options:**
A. az sql db create
B. az sql db op
C. az sql db import
D. az sql db copy

**✅ Answer:** C

**Why:** `az sql db import` "imports a bacpac into a new database, or an existing empty database" (from Azure Storage via --storage-uri). `create` makes an empty DB, `op` manages running operations, and `copy` copies an existing database, not a BACPAC.

**Source-answer check:** MD said C ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/cli/azure/sql/db?view=azure-cli-latest

---

### Q212. Bicep server + Database1 statements
<sub>ID U-226</sub>

**Sources:** MD-072   **Type:** Statements(Yes/No)   **Case study:** —

**Question:**
You have an Azure subscription. You deploy resources by using the following Bicep template.
```
resource sqlServer 'Microsoft.Sql/server@2022-05-01-preview' = {
     name: 'Server1'
     location: 'Sweden central'
     properties: {
          administrators: {
               login: sqladmins
               principalType: 'Group'
               administratorType: 'ActiveDirectory'
          }
     }
}

resource sqlDB 'Microsoft.Sql/servers/databases@2022-05-01-preview' = {
     parent: 'Server1'
     name: 'Database1'
     location: 'Sweden central'
     sku: {
          name: 'Standard'
          tier: 'Standard'
     }
}
```
For each of the following statements, select Yes if the statement is true. Otherwise, select No.

**Options:**
1. Server1 will support mixed authentication.
2. Database1 will scale dynamically based on the load.
3. Server1 will be created as an Azure SQL managed instance.

**✅ Answer:** 1: Yes; 2: No; 3: No

**Why:** (1) An Entra group admin is set but `azureADOnlyAuthentication` isn't specified. Its default is false, so SQL authentication stays enabled alongside Entra authentication (mixed). (2) sku Standard/Standard is the DTU-model provisioned Standard tier with fixed compute. Only vCore serverless scales compute automatically. (3) The resource type is the logical server (Microsoft.Sql/servers), not Microsoft.Sql/managedInstances. Exam-intent note: in a real deployment, leaving Entra-only auth false also requires administratorLogin/password, which this template omits. That doesn't change the intended answers.

**Source-answer check:** MD said Yes/No/No ✔
**Confidence:** Medium (statement 1 relies on the default azureADOnlyAuthentication=false; the template as written is incomplete)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-only-authentication-create-server?view=azuresql

---

### Q213. New-AzSqlDatabase: ComputeGeneration and ComputeModel
<sub>ID U-227</sub>

**Sources:** MD-059   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure subscription. You need to deploy an Azure SQL database. The solution must meet the following requirements:
- Support at least 36 cores.
- Minimize costs.

How should you complete the PowerShell script?
```
$SqlDatabaseParams = @{
  ResourceGroupName = "RG1"
  ServerName = "Server01"
  DatabaseName = "DB1"
  Edition = "GeneralPurpose"
  Vcore = 2
  ComputeGeneration = [Box1]
  ComputeModel = [Box2]
}
New-AzSqlDatabase @SqlDatabaseParams
```

**Options:**
Box 1 (ComputeGeneration): "4" | "5" | "Gen4" | "Gen5"
Box 2 (ComputeModel): Free | Provisioned | Serverless | Standard

**✅ Answer:** Box 1: "Gen5"; Box 2: Serverless

**Why:** ComputeGeneration takes family names ("Gen4"/"Gen5"), not "4"/"5". Gen4 is retired and topped out at 24 vCores, so Gen5 (standard-series) is needed for 36+ cores. ComputeModel accepts only Provisioned or Serverless. Serverless General Purpose Gen5 scales up to 80 vCores and bills per second for compute used (with auto-pause), so it minimizes cost for the requirement. The cmdlet's own serverless example is `-Edition GeneralPurpose -Vcore 2 -ComputeGeneration "Gen5" -ComputeModel Serverless`. Free and Standard aren't ComputeModel values.

**Source-answer check:** MD said Gen5 / Serverless ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/powershell/module/az.sql/new-azsqldatabase

---

### Q214. Change SQL MI to Business Critical
<sub>ID U-228</sub>

**Sources:** MD-068   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription that contains an Azure SQL managed instance named SQLMI1. You need to configure SQLMI1 to use the Business Critical service tier. Which PowerShell cmdlet should you run?

**Options:**
A. Update-SqlVM
B. Set-AzSqlServerConfigurationOption
C. Set-AzSqlDatabase
D. Set-AzSqlInstance

**✅ Answer:** D

**Why:** Set-AzSqlInstance modifies an existing managed instance, including `-Edition BusinessCritical` (service tier), vCores and storage. Set-AzSqlDatabase changes a single database on a logical server. Set-AzSqlServerConfigurationOption sets MI server configuration options (e.g., allowPolybaseExport), not the tier. Update-SqlVM is for SQL Server on Azure VMs.

**Source-answer check:** MD said D ✔
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/powershell/module/az.sql/set-azsqlinstance (Example 1: `-Edition BusinessCritical`)

---

### Q215. ADatum: first step for DB3 elastic jobs
<sub>ID U-229</sub>

**Sources:** MD-075   **Type:** MCQ   **Case study:** ADatum v1 (financial)

**Question:**
(Case study — ADatum Corporation v1. DB3 is migrated from SQL Server 2019 on SVR3 to Azure SQL Database. Management requirement: "In Azure SQL Database, automate the management of DB3 by using elastic jobs that have database-scoped credentials." Business requirements: minimize costs and administrative effort.)
You need to recommend a process to automate the management of DB3. The solution must meet the management requirements. What should be the first step of the process?

**Options:**
A. Configure Microsoft Entra authentication for the logical server that hosts DB3.
B. Configure a private endpoint for connectivity to DB3.
C. Create database-scoped credentials in DB3.
D. Create a database that has database-scoped credentials.

**✅ Answer:** D

**Why:** An elastic job agent can only be created on an existing Azure SQL database, which becomes the *job database* (a new, empty database, S1 or higher recommended). With database-scoped credential authentication, Microsoft's docs say to "create the database-scoped credentials in the *job database*". The target (DB3) only needs a login/user (jobuser) whose name and password match the credential. So the process starts by creating the job database that holds the credentials, then the agent, target group and jobs. C puts the credentials in the target, which is only needed when the target is itself the job database. A is the UMI/Entra method, not the required database-scoped credentials. B is optional networking.

**Source-answer check:** MD said C ✘ — elastic-job credentials live in the job database, not in the target DB3. C works only if DB3 itself were the job database, which the docs advise against (the job database should be new and empty).
**Confidence:** Medium (no community key found; answer derived from the elastic jobs docs)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-overview?view=azuresql

---

### Q216. Elastic jobs auth with Conditional Access
<sub>ID U-230</sub>

**Sources:** MD-132   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription that contains an Azure SQL database named DB1. You need to host elastic jobs by using DB1. DB1 will also be configured as a job target. The solution must support the use of location-based Conditional Access policies. What should the elastic jobs use to access DB1?

**Options:**
A. a system-assigned managed identity
B. Azure SQL sign-in credentials
C. database-scoped credentials
D. a user-assigned managed identity

**✅ Answer:** D

**Why:** Elastic jobs support exactly two target-authentication methods. One is Microsoft Entra authentication with a user-assigned managed identity, which Microsoft recommends. The other is database-scoped credentials (SQL authentication). A system-assigned managed identity is explicitly not supported (A ✘), and B isn't an elastic-job auth method. Conditional Access, including location-based policies, applies only to Microsoft Entra authentication. SQL-auth credentials (C) bypass Entra entirely, so they can't be governed by CA. That leaves the UMI. Caveat: Entra's "Conditional Access for workload identities" doesn't currently cover managed identities, so a CA policy wouldn't actually evaluate the UMI sign-in. Even under the reading "must keep working where CA policies are in force", the UMI works and is the recommended option.

**Source-answer check:** MD said C ✘ — MD argues CA can't apply to managed identities, which rules out A/D. That argument applies equally to database-scoped credentials (SQL auth is never subject to CA), so it doesn't single out C. The docs-based reading (CA ⇒ Entra auth ⇒ UMI, since SMI is unsupported) gives D.
**Confidence:** Low (genuinely ambiguous wording; dump key is C, docs-based reasoning favors D)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-overview?view=azuresql ; https://learn.microsoft.com/entra/identity/conditional-access/workload-identity

---

### Q217. Minimum database scoped credentials for elastic jobs
<sub>ID U-231</sub>

**Sources:** MD-126   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure subscription that contains the resources shown in the following table.

| Name | Type | Description |
|---|---|---|
| SQLSVR1 | Logical SQL server | Hosts one Azure SQL database |
| SQLDB11 | Azure SQL database | Is hosted by SQLSVR1 |
| SQLSVR2 | Logical SQL server | Hosts two Azure SQL databases |
| SQLDB21 | Azure SQL database | Is hosted by SQLSVR2 |
| SQLDB22 | Azure SQL database | Is hosted by SQLSVR2 |

You plan to use SQLDB11 as an elastic job database to run jobs on SQLDB11 and SQLDB22. What is the minimum number of database scoped credentials required for the elastic jobs?

**Options:**
A. 1
B. 2
C. 3
D. 4

**✅ Answer:** A

**Why:** All elastic-job credentials are created in the job database (SQLDB11). The targets are two individual databases, so they can go in a target group as single-database members, which needs only the job-execution credential (jobuser). Create a matching jobuser login/user with the same password in SQLDB11 and SQLDB22, and one credential covers both. A second "master" (refresh) credential is required only when a target group contains a *server* or *elastic pool*, because the agent must connect to master to enumerate databases. Targeting just two named databases doesn't need it. The agent reaches its own job database with internal certificate auth, so that needs no credential either.

**Source-answer check:** MD said B (jobuser + masteruser) ✘ — the master credential is only needed for server/pool target-group members. The "minimum" here is 1. Community votes on public dump sites are scattered (1/2/3), so this one is contested.
**Confidence:** Medium (doc rule is clear; exam key may still be B)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-overview?view=azuresql

---


## D5 — Plan and configure a high availability and disaster recovery (HA/DR) environment (20–25%)

*13 questions*

### Q218. BACKUP LOG NORECOVERY COPY_ONLY statements
<sub>ID U-232</sub>

**Sources:** NOTE-241   **Type:** Statements(Yes/No)   **Case study:** —

**Question:**
You have a SQL Server on Azure Virtual Machines instance named VM1 that hosts a database named DB1. You run the following query.
```sql
BACKUP LOG DB1 TO DISK = '\File1\SQLBackups\DB1.trn'
WITH NORECOVERY, COPY_ONLY, CONTINUE_AFTER_ERROR;
GO
```
For each of the following statements, select Yes if the statement is true. Otherwise, select No.

**Options:**
1. The log file will be truncated.
2. DB1 will be placed in an offline state.
3. You are performing a tail-log backup.

**✅ Answer:** 1. No; 2. No; 3. Yes

**Why:** Per the tail-log backup doc, a BACKUP LOG ... WITH NORECOVERY "takes the database into the restoring state" and "the log is truncated unless the NO_TRUNCATE option or COPY_ONLY option is also specified", so COPY_ONLY means no truncation. The database goes to RESTORING, which is a different state from OFFLINE. NORECOVERY together with CONTINUE_AFTER_ERROR (which the doc says is only for backing up the tail of a damaged database) is a tail-log backup.

**Source-answer check:** The NOTE hand-mark said Yes/No/No. Statement 1 is wrong ✘ because COPY_ONLY prevents truncation. Statement 2 (No) is correct ✔. Statement 3 is wrong ✘ because WITH NORECOVERY on a log backup is a tail-log backup.
**Confidence:** Medium (statement 2 depends on reading "offline" strictly: the database is unavailable but in the RESTORING state, not OFFLINE)   **Ref:** https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/tail-log-backups-sql-server

---

### Q219. LTR policy retention of weekly backups
<sub>ID U-233</sub>

**Sources:** NOTE-img033   **Type:** Hotspot   **Case study:** —

**Question:** (PARTIAL. The header and the LTR policy exhibit were not captured, and the full version could not be found online. Best reconstruction: an Azure SQL database has a long-term retention policy, shown in an exhibit, that offers retention periods of 6 weeks, 12 months and 10 years. This most likely matches the Microsoft example W=6 weeks, M=12 months, Y=10 years with a WeekOfYear that is not week 1 or week 2.)
The first weekly backup occurred on January 4, 2020. The dates for the first 10 weekly backups are: January 4, January 11, January 18, January 25, February 1, February 8, February 15, February 22, February 29 and March 7, 2020.
Use the drop-down menus to select the answer choice that completes each statement based on the information presented in the graphic.

**Options:**
Statement 1: The backup saved to long-term retention on January 4, 2020, will be retained for: 6 weeks / 12 months / 10 years
Statement 2: The backup saved to long-term retention on January 11, 2020, will be retained for: 6 weeks / 12 months / 10 years

**✅ Answer (best reconstruction):** Statement 1: 12 months; Statement 2: 6 weeks

**Why:** LTR rules: with W set, every weekly full backup is kept for W. With M set, *the first full backup of each month* is kept for M. With Y set, the backup in week WeekOfYear is kept for Y, and the longest applicable tier wins (doc example "W=6, M=12, Y=10…"). January 4 is the first backup of January, so it is the monthly backup and is kept 12 months. January 11 is an ordinary weekly backup and is kept 6 weeks. Caveat: if the exhibit's WeekOfYear is 1, January 4 is kept 10 years; if it is 2, January 11 is kept 10 years.

**Source-answer check:** The NOTE hand-mark said Jan 4 = 6 weeks and Jan 11 = 12 months ✘. That looks swapped: under LTR rules the first backup of the month, not the second, gets the monthly retention.
**Confidence:** Low (the exhibit is missing; the answer is inferred from the documented LTR selection rules and the answer-choice set)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/long-term-retention-overview

---

### Q220. Managed instance BACKUP TO URL COPY_ONLY
<sub>ID U-234</sub>

**Sources:** NOTE-001   **Type:** Hotspot   **Case study:** —

**Question:**
You have an Azure SQL Database managed instance named sqldbmi1 that contains a database named Sales. You need to initiate a backup of Sales. How should you complete the Transact-SQL statement?
```
BACKUP DATABASE Sales
[Box 1]
WITH STATS = 5,
[Box 2]
```

**Options:**
Box 1: TO DISK = '\BackupSystem\BackupDisk1\Sales.bak' / TO DISK = 'X:\BAK\Sales.bak' / TO 'Sales_Backup' / TO URL = 'https://storage1.blob.core.windows.net/blob1/Sales.bak'
Box 2: COPY_ONLY / ENCRYPTION / FILE_SNAPSHOT / NO_TRUNCATE

**✅ Answer:** Box 1: TO URL = 'https://storage1.blob.core.windows.net/blob1/Sales.bak'; Box 2: COPY_ONLY

**Why:** SQL Managed Instance has no access to local or UNC disks or backup devices, so user backups can only go to Azure Blob storage with TO URL. Automated backups own the backup chain, so a user-initiated backup must be COPY_ONLY. FILE_SNAPSHOT needs database files that are stored in Azure Blob, ENCRYPTION needs a certificate or key and is not required here, and NO_TRUNCATE applies only to log backups.

**Source-answer check:** The NOTE answer is correct ✔ (URL + COPY_ONLY).
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/automated-backups-overview

---

### Q221. PowerShell STR 21 days and LTR statements
<sub>ID U-235</sub>

**Sources:** NOTE-39   **Type:** Statements(Yes/No)   **Case study:** —

**Question:**
You have an Azure SQL database. You run the following PowerShell script.
```powershell
$serverName = "SERVER1"
$resourceGroup = "RG1"
$dbName = "DB1"
Connect-AzAccount
$server = Get-AzSqlServer -ServerName $serverName -ResourceGroupName $resourceGroup
Set-AzSqlDatabaseBackupShortTermRetentionPolicy -ResourceGroupName $resourceGroup -ServerName $server `
    -DatabaseName $dbName -RetentionDays 21
Set-AzSqlDatabaseBackupLongTermRetentionPolicy -ServerName $serverName -DatabaseName $dbName `
    -ResourceGroupName $resourceGroup -WeeklyRetention P52W -YearlyRetention P5Y -WeekOfYear 52
```
For each statement, select Yes if true; otherwise, select No.

**Options:**
1. DB1 can be restored to a specific point in time 30 days ago.
2. DB1 can be restored from a weekly backup performed six months ago.
3. DB1 can be restored from a yearly backup performed six years ago.

**✅ Answer:** 1. No; 2. Yes; 3. No

**Why:** Short-term (PITR) retention is 21 days, so a point 30 days ago is outside the window. Weekly LTR backups are kept 52 weeks, so a weekly backup from 6 months ago still exists. Yearly backups (week 52) are kept 5 years, so one from 6 years ago has expired.

**Source-answer check:** The NOTE answer (No/Yes/No) is correct ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/long-term-retention-overview

---

### Q222. Restore sequence full, differential, logs
<sub>ID U-236</sub>

**Sources:** NOTE-111   **Type:** DragDrop   **Case study:** —

**Question:**
You have SQL Server on an Azure virtual machine that contains a database named DB1. DB1 is 30 TB and has a 1-GB daily rate of change. You back up the database by using a Microsoft SQL Server Agent job that runs Transact-SQL commands. You perform a weekly full backup on Sunday, daily differential backups at 01:00, and transaction log backups every five minutes. The database fails on Wednesday at 10:00. Which three backups should you restore in sequence?

**Options:**
- Monday, Tuesday, and then Wednesday differential backups
- Wednesday, Tuesday, and then Monday log backups
- full backup
- Monday, Tuesday, and then Wednesday log backups
- Wednesday, Tuesday, and then Monday differential backups
- Wednesday log backups
- Wednesday differential backup

**✅ Answer:** 1. full backup → 2. Wednesday differential backup → 3. Wednesday log backups

**Why:** Differential backups are cumulative since the last full backup, so only the latest one (Wednesday 01:00) is needed. After it, restore the log backups taken after 01:00 Wednesday, in order, up to the failure. Restoring the Monday and Tuesday differentials or logs is unnecessary, and reverse order is invalid.

**Source-answer check:** The NOTE answer is correct ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/cs-cz/sql/relational-databases/backup-restore/restore-a-differential-database-backup-sql-server?view=sql-server-2016

---

### Q223. DR test failover for Hyper-V app
<sub>ID U-237</sub>

**Sources:** NOTE-21-img037   **Type:** MCQ   **Case study:** —

**Question:**
You have an on-premises multi-tier application named App1 that includes a web tier, an application tier, and a Microsoft SQL Server tier. All the tiers run on Hyper-V virtual machines. Your new disaster recovery plan requires that all business-critical applications can be recovered to Azure. You need to recommend a solution to fail over the database tier of App1 to Azure. The solution must provide the ability to test failover to Azure without affecting the current environment. What should you include in the recommendation?

**Options:**
A. Azure Backup
B. Azure Information Protection
C. Windows Server Failover Cluster
D. Azure Site Recovery

**✅ Answer:** D

**Why:** Azure Site Recovery replicates Hyper-V VMs to Azure and supports *test failover* into an isolated network with no impact on production replication or workloads. Azure Backup is for backup and restore, not orchestrated failover. AIP is data classification. A WSFC alone does not give a non-disruptive DR drill to Azure.

**Source-answer check:** The NOTE answer is correct ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-test-failover-to-azure

---

### Q224. Litware ManufacturingSQLDb1 witness and listener
<sub>ID U-238</sub>

**Sources:** DEMO-5, NOTE-289   **Type:** Hotspot   **Case study:** Litware

**Question:**
You need to recommend a configuration for ManufacturingSQLDb1 after the migration to Azure. The solution must meet the business requirements. What should you include in the recommendation?
(Litware: ManufacturingSQLDb1 is in an on-premises Always On AG on SERVER2/SERVER3 and will be migrated to Azure VMs. Business requirement: "Meet an SLA of 99.99% availability for all Azure deployments"; minimize costs.)

**Options:**
Quorum model: Cloud witness / Disk witness / File share witness
Azure resource for the availability group listener: Azure Application Gateway / Azure Basic Load Balancer

**✅ Answer:** Quorum model: Cloud witness; AG listener: Azure Basic Load Balancer (the only valid listener choice offered)

**Why:** For a failover cluster on Azure VMs whose nodes can reach the internet, Microsoft recommends a Cloud Witness (Azure Storage). It needs no extra VM, unlike a file share witness, and shared disks aren't needed. A VNN AG listener on Azure VMs needs an Azure Load Balancer. Application Gateway is an L7 HTTP proxy and can't front a SQL listener. Note on current docs: the Basic Load Balancer SKU was retired on 30 Sep 2025, and Microsoft now recommends a Standard Load Balancer (or a DNN listener or multi-subnet AG). If a "Standard Load Balancer" option appears, pick it. The demo's own explanation cites the Standard LB 99.99% SLA.

**Source-answer check:** DEMO-5 and NOTE-289 both say Cloud witness + Basic Load Balancer ✔ (correct for the options shown; the exam-era key).
**Confidence:** Medium (correct among the listed options; Basic LB is now retired)   **Ref:** https://learn.microsoft.com/en-us/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener

---

### Q225. Zone-redundant automatic failover at minimum cost
<sub>ID U-239</sub>

**Sources:** NOTE-083   **Type:** MCQ   **Case study:** —

**Question:**
You have an Azure SQL database named DB1. You need to ensure that DB1 will support automatic failover without data loss if a datacenter fails. The solution must minimize costs. Which deployment option and pricing tier should you configure?

**Options:**
A. Azure SQL Database Premium
B. Azure SQL Database serverless
C. Azure SQL Database managed instance Business Critical
D. Azure SQL Database Standard

**✅ Answer:** A

**Why:** Surviving a datacenter (availability zone) failure with automatic failover and zero data loss needs zone redundancy. In the DTU model only Premium supports zone redundancy, and Standard and Basic do not. Zone redundancy in Premium and Business Critical comes at no extra cost because it uses the existing replicas. MI Business Critical is more expensive. Current-docs caveat: General Purpose (including serverless) now also supports zone redundancy, which makes B arguable today. The exam key, written when serverless and General Purpose zone redundancy weren't available, is A.

**Source-answer check:** The NOTE answer (A) is correct ✔ per the exam key.
**Confidence:** Medium (B is defensible under current docs, because General Purpose serverless now supports zone redundancy)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/database/high-availability-sla-local-zone-redundancy

---

### Q226. Failover without connection string changes
<sub>ID U-240</sub>

**Sources:** NOTE-44   **Type:** MCQ   **Case study:** —

**Question:**
You need to recommend an availability strategy for an Azure SQL database. The strategy must meet the following requirements:
- Support failovers that do not require client applications to change their connection strings.
- Replicate the database to a secondary Azure region.
- Support failover to the secondary region.
What should you include in the recommendation?

**Options:**
A. failover groups
B. transactional replication
C. Availability Zones
D. geo-replication

**✅ Answer:** A

**Why:** Failover groups replicate to another region and expose read-write and read-only listener endpoints whose DNS follows the primary, so connection strings don't change after failover. Active geo-replication alone needs apps to point at the new server name. Availability Zones stay within one region. Transactional replication has no automatic failover or endpoint redirection.

**Source-answer check:** The NOTE answer is correct ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/azure/azure-sql/database/auto-failover-group-sql-db

---

### Q227. Log full due to AVAILABILITY_REPLICA fix
<sub>ID U-241</sub>

**Sources:** MD-194   **Type:** DragDrop   **Case study:** —

**Question:**
You have a database named db1. The log for db1 contains the following entry.
```
Date 10/5/2021 10:57:08 AM
Log SQL Server (Current - 10/5/2021 11:26:00 AM)
Sources spid1595
Message
The transaction log for database 'db1' is full due to 'AVAILABILITY_REPLICA'
```
You need to ensure that db1 can process transactions. Which three actions should you perform in sequence?

**Options:**
- Back up the transaction log file.
- Remove db1 from the availability group.
- Shrink the transaction log file.
- Shrink db1.
- Add db1 back to the availability group.

**✅ Answer:** 1. Remove db1 from the availability group → 2. Shrink the transaction log file → 3. Add db1 back to the availability group

**Why:** Error 9002 with AVAILABILITY_REPLICA means log records haven't been hardened or redone on a secondary, so the primary can't truncate the log. A log backup doesn't free the space while the secondary lags. Microsoft's workaround is to remove the database from the AG, which immediately allows truncation, then shrink the log file and re-add the database (re-seeding the secondary). Shrinking the whole database (data files) doesn't address the log.

**Source-answer check:** The MD answer is correct ✔.
**Confidence:** High   **Ref:** https://learn.microsoft.com/en-us/troubleshoot/sql/availability-groups/error-9002-transaction-log-large

---

### Q228. MI failover group automatic policy results
<sub>ID U-242</sub>

**Sources:** NOTE-121   **Type:** MultiSelect   **Case study:** —

**Question:**
You are planning disaster recovery for the failover group of an Azure SQL Database managed instance. Your company's SLA requires that the database in the failover group become available as quickly as possible if a major outage occurs. You set the Read/Write failover policy to Automatic. What are two results of the configuration? Each correct answer presents a complete solution.

**Options:**
A. In the event of a datacenter or Azure regional outage, the databases will fail over automatically.
B. In the event of an outage, the databases in the primary instance will fail over immediately.
C. In the event of an outage, you can selectively fail over individual databases.
D. In the event of an outage, you can set a different grace period to fail over each database.
E. In the event of an outage, the minimum delay for the databases to fail over in the primary instance will be one hour.

**✅ Answer:** A, E

**Why:** With the Automatic read/write failover policy (called "Microsoft managed" in the current portal), Azure starts a geo-failover after an outage once the grace period (GracePeriodWithDataLossHours) passes. The minimum grace period is 1 hour, which is also the default, so failover isn't immediate. A managed-instance failover group fails over all user databases together, with one grace period for the whole group, so C and D are false.

**Source-answer check:** NOTE marked A, B. A is correct ✔. B is wrong ✘: failover waits for at least the 1-hour grace period, so the correct second answer is E.
**Confidence:** High   **Ref:** https://learn.microsoft.com/powershell/module/az.sql/new-azsqldatabaseinstancefailovergroup ; https://learn.microsoft.com/en-us/training/modules/explore-iaas-paas-platform-tools-for-high-availability-disaster-recovery/6-explore-auto-failover-groups

---

### Q229. Prepare AG secondary with RESTORE NORECOVERY
<sub>ID U-243</sub>

**Sources:** NOTE-219   **Type:** Hotspot   **Case study:** —

**Question:**
You have two Azure virtual machines named VM1 and VM2 that run Windows Server 2019. VM1 and VM2 each host a default Microsoft SQL Server 2019 instance. VM1 contains a database named DB1 that is backed up to a file named D:\DB1.bak. You plan to deploy an Always On availability group with VM1 hosting the primary replica of DB1 and VM2 hosting a secondary replica of DB1. You need to prepare the secondary database on VM2 for the availability group. How should you complete the Transact-SQL statement?
```
[Box 1] DATABASE MyDB1
  FROM DISK = 'D:\DB1.bak'
  WITH [Box 2]
GO
```

**Options:**
Box 1: BACKUP / CREATE / RESTORE
Box 2: NORECOVERY / RECOVERY / STANDBY

**✅ Answer:** Box 1: RESTORE; Box 2: NORECOVERY

**Why:** A secondary database is prepared by restoring the primary's full backup (and later log backups) on the secondary instance WITH NORECOVERY, which leaves it in the RESTORING state so it can be joined to the AG. RECOVERY brings the database online and breaks the log chain needed to join. STANDBY (read-only standby) is for log shipping, not AG joins.

**Source-answer check:** The NOTE hand-mark of RESTORE is correct ✔. Its RECOVERY mark is wrong ✘: the secondary must be restored WITH NORECOVERY.
**Confidence:** High   **Ref:** https://learn.microsoft.com/En-Us/Sql/database-engine/availability-groups/windows/manually-prepare-a-secondary-database-for-an-availability-group-sql-server?view=aps-pdw-2016

---

### Q230. User backups plus multi-region replicas
<sub>ID U-244</sub>

**Sources:** NOTE-46   **Type:** Hotspot   **Case study:** —

**Question:**
You plan to migrate on-premises Microsoft SQL Server databases to Azure. You need to identify which deployment and resiliency options meet the following requirements:
- Support user-initiated backups.
- Support multiple automatically replicated instances across Azure regions.
- Minimize administrative effort to implement and maintain business continuity.
What should you identify?

**Options:**
Deployment option: Azure SQL Managed Instance / SQL Server on Azure Virtual Machines / An Azure SQL Database single database
Resiliency option: Auto-failover group / Active geo-replication / Zone-redundant deployment

**✅ Answer:** Deployment option: Azure SQL Managed Instance; Resiliency option: Auto-failover group

**Why:** SQL Managed Instance supports user-initiated (COPY_ONLY to URL) backups, which a single Azure SQL Database does not, and it is PaaS, so business continuity takes far less effort than on SQL Server on VMs (where you build and run AGs and clusters yourself). For MI, cross-region automatic replication is provided by auto-failover groups. Active geo-replication isn't available for MI, and zone redundancy stays within one region.

**Source-answer check:** The NOTE hand-mark said SQL Server on Azure VMs ✘ (VMs need the most administrative effort for HA/DR). Its Auto-failover group mark is correct ✔.
**Confidence:** Medium (community consensus; docs confirm each component)   **Ref:** https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/automated-backups-overview ; https://learn.microsoft.com/ar-sa/azure/azure-sql/managed-instance/failover-group-sql-mi

---

## Case studies

Questions marked **Case study: X** refer to the text below.

One section per distinct case study. Each section uses the most complete text found in the sources; exact duplicates are removed.
Text versions compared: Litware, ADatum v1 and Contoso appear in DP-300.md testlets (Q1-56, Q57-112, Q113-168). They differ only in line breaks and heading placement. The Q57-112 transcription is used because it has the "X identifies the following..." lead-ins and the most detailed exhibit tables.
The CertyIQ PDF, the demo PDF and the notes screenshots contain no case-study text. Their case questions are mapped to these cases by entity names.

---

### Litware, Inc. (renewable energy; SERVER1, SalesSQLDb1, ResearchDB1/ResearchApp1, ManufacturingSQLDb1)

Questions: Q8 (MD-001, DEMO-1); Q68 (MD-078, NOTE-131); Q78 (MD-079, DEMO-2, NOTE-296); Q154 (MD-149, DEMO-3); Q155 (DEMO-4); Q205 (NOTE-img059); Q224 (DEMO-5, NOTE-289)

Note: in the source, the licensing sentence sits under the "Current Problems" heading. It is kept there as in the original.

Overview
Litware, Inc. is a renewable energy company that has a main office in Boston. The main office hosts a sales department and the primary datacenter for the company.

Physical Locations
Litware has a manufacturing office and a research office in separate locations near Boston. Each office has its own datacenter and internet connection.

Existing Environment

Network Environment
The manufacturing and research datacenters connect to the primary datacenter by using a VPN.
The primary datacenter has an ExpressRoute connection that uses both Microsoft peering and private peering. The private peering connects to an Azure virtual network named HubVNet.

Identity Environment
Litware has a hybrid Microsoft Entra ID deployment that uses a domain named litwareinc.com. All Azure subscriptions are associated to the litwareinc.com Microsoft Entra tenant.

Database Environment
The sales department has the following database workload:
- An on-premises server named SERVER1 hosts an instance of Microsoft SQL Server 2012 and two 1-TB databases.
- A logical server named SalesSrv01A contains a geo-replicated Azure SQL database named SalesSQLDb1. SalesSQLDb1 is in an elastic pool named SalesSQLDb1Pool. SalesSQLDb1 uses database firewall rules and contained database users.
- An application named SalesSQLDb1App1 uses SalesSQLDb1.
The manufacturing office contains two on-premises SQL Server 2016 servers named SERVER2 and SERVER3. The servers are nodes in the same Always On availability group. The availability group contains a database named ManufacturingSQLDb1.
Database administrators have two Azure virtual machines in HubVnet named VM1 and VM2 that run Windows Server 2019 and are used to manage all the Azure databases.

Current Problems
Litware is a Microsoft Volume Licensing customer that has License Mobility through Software Assurance.
SalesSQLDb1 experiences performance issues that are likely due to out-of-date statistics and frequent blocking queries.

Requirements

Planned Changes
Litware plans to implement the following changes:
- Implement 30 new databases in Azure, which will be used by time-sensitive manufacturing apps that have varying usage patterns. Each database will be approximately 20 GB.
- Create a new Azure SQL database named ResearchDB1 on a logical server named ResearchSrv01. ResearchDB1 will contain Personally Identifiable Information (PII) data.
- Develop an app named ResearchApp1 that will be used by the research department to populate and access ResearchDB1.
- Migrate ManufacturingSQLDb1 to the Azure virtual machine platform.
- Migrate the SERVER1 databases to the Azure SQL Database platform.

Technical Requirements
Litware identifies the following technical requirements:
- Maintenance tasks must be automated.
- The 30 new databases must scale automatically.
- The use of an on-premises infrastructure must be minimized.
- Azure Hybrid Use Benefits must be leveraged for Azure SQL Database deployments.
- All SQL Server and Azure SQL Database metrics related to CPU and storage usage and limits must be analyzed by using Azure built-in functionality.

Security and Compliance Requirements
Litware identifies the following security and compliance requirements:
- Store encryption keys in Azure Key Vault.
- Retain backups of the PII data for two months.
- Encrypt the PII data at rest, in transit, and in use.
- Use the principle of least privilege whenever possible.
- Authenticate database users by using Active Directory credentials.
- Protect Azure SQL Database instances by using database-level firewall rules.
- Ensure that all databases hosted in Azure are accessible from VM1 and VM2 over the Azure backbone network.

Business Requirements
Litware identifies the following business requirements:
- Meet an SLA of 99.99% availability for all Azure deployments.
- Minimize downtime during the migration of the SERVER1 databases.
- Use the Azure Hybrid Use Benefits when migrating workloads to Azure.
- Once all requirements are met, minimize costs whenever possible.

---

### ADatum Corporation v1 (financial services; SVR1-3, DB1/DB2/DB3, Instance1)

Questions: Q7 (MD-002); Q44 (MD-003); Q77 (MD-076); Q91 (MD-146); Q108 (MD-077); Q143 (MD-148); Q176 (MD-147); Q215 (MD-075)

Overview
ADatum Corporation is a financial services company that has a main office in New York City.

Existing Environment. Licensing Agreement
ADatum has a Microsoft Volume Licensing agreement that includes Software Assurance.

Existing Environment. Network Infrastructure
ADatum has an on-premises datacenter and an Azure subscription named Sub1.
Sub1 contains a virtual network named Network1 in the East US Azure region.
The datacenter is connected to Network1 by using a Site-to-Site (S2S) VPN.

Existing Environment. Identity Environment
The on-premises network contains an Active Directory Domain Services (AD DS) forest.
The forest contains a single domain named corp.adatum.com.
The corp.adatum.com domain syncs with a Microsoft Entra tenant named adatum.com.

Existing Environment. Database Environment
The datacenter contains the servers shown in the following table.

| Name | Operating system | Description |
|---|---|---|
| SVR1 | Windows Server 2016 | Has Microsoft SQL Server 2016 Enterprise installed. Has an Always On availability group named AG1 that contains two databases named DB1 and DB2. |
| SVR2 | Windows Server 2016 | Has Microsoft SQL Server 2016 Enterprise installed. Has an Always On availability group named AG1 that contains two databases named DB1 and DB2. |
| SVR3 | Windows Server 2019 | Has Microsoft SQL Server 2019 Enterprise installed that contains a database named DB3. |

DB1 and DB2 are used for transactional and analytical workloads by an application named App1.
App1 runs on Microsoft Entra hybrid joined servers that run Windows Server 2022. App1 uses Kerberos authentication.
DB3 stores compliance data used by two applications named App2 and App3.
DB3 performance is monitored by using Extended Events sessions, with the event_file target set to a file share on a local disk of SVR3.
Resource allocation for DB3 is managed by using Resource Governor.

Requirements. Planned Changes
ADatum plans to implement the following changes:
- Deploy an Azure SQL managed instance named Instance1 to Network1.
- Migrate DB1 and DB2 to Instance1.
- Migrate DB3 to Azure SQL Database.
- Following the migration of DB1 and DB2, hand over database development to remote developers who use Microsoft Entra joined Windows 11 devices.
- Following the migration of DB3, configure the database to be part of an auto-failover group.

Requirements. Availability Requirements
ADatum identifies the following post-migration availability requirements:
- For DB1 and DB2, offload analytical workloads to a read-only database replica in the same Azure region.
- Ensure that if a regional disaster occurs, DB1 and DB2 can be recovered from backups.
- After the migration, App1 must maintain access to DB1 and DB2.
- For DB3, manage potential performance issues caused by resource demand changes by App2 and App3.
- Ensure that DB3 will still be accessible following a planned failover.
- Ensure that DB3 can be restored if the logical server is deleted.
- Minimize downtime during the migration of DB1 and DB2.

Requirements. Security Requirements
ADatum identifies the following security requirements for after the migration:
- Ensure that only designated developers who use Microsoft Entra joined Windows 11 devices can access DB1 and DB2 remotely.
- Ensure that all changes to DB3, including ones within individual transactions, are audited and recorded.

Requirements. Management Requirements
ADatum identifies the following post-migration management requirements:
- Continue using Extended Events to monitor DB3.
- In Azure SQL Database, automate the management of DB3 by using elastic jobs that have database-scoped credentials.

Requirements. Business Requirements
ADatum identifies the following business requirements:
- Minimize costs whenever possible, without affecting other requirements.
- Minimize administrative effort.

---

### Contoso, Ltd. (financial data; one Azure SQL database per customer)

Questions: Q45 (MD-150); Q74 (MD-151); Q115 (MD-081, NOTE-127); Q116 (MD-080); Q123 (MD-082)

Overview

General Overview
Contoso, Ltd. is a financial data company that has 100 employees. The company delivers financial data to customers.

Physical Locations
Contoso has a datacenter in Los Angeles and an Azure subscription. All Azure resources are in the US West 2 Azure region. Contoso has a 10-Gb ExpressRoute connection to Azure.
The company has customers worldwide.

Existing Environment

Active Directory
Contoso has a hybrid Microsoft Entra ID deployment that syncs to on-premises Active Directory.

Database Environment
Contoso has SQL Server 2017 on Azure virtual machines shown in the following table.

| Name | Role |
|---|---|
| SQL1 | Primary data warehouse |
| SQL2 | Secondary data warehouse |
| SQL3 | Extract, transform, and load (ETL) server |

SQL1 and SQL2 are in an Always On availability group and are actively queried. SQL3 runs jobs, provides historical data, and handles the delivery of data to customers.
The on-premises datacenter contains a PostgreSQL server that has a 50-TB database.

Current Business Model
Contoso uses Microsoft SQL Server Integration Services (SSIS) to create flat files for customers. The customers receive the files by using FTP.

Requirements

Planned Changes
Contoso plans to move to a model in which they deliver data to customer databases that run as platform as a service (PaaS) offerings.
When a customer establishes a service agreement with Contoso, a separate resource group that contains an Azure SQL database will be provisioned for the customer. The database will have a complete copy of the financial data. The data to which each customer will have access will depend on the service agreement tier. The customers can change tiers by changing their service agreement.
The estimated size of each PaaS database is 1 TB.
Contoso plans to implement the following changes:
- Move the PostgreSQL database to Azure Database for PostgreSQL during the next six months.
- Upgrade SQL1, SQL2, and SQL3 to SQL Server 2019 during the next few months.
- Start onboarding customers to the new PaaS solution within six months.

Business Goals
Contoso identifies the following business requirements:
- Use built-in Azure features whenever possible.
- Minimize development effort whenever possible.
- Minimize the compute costs of the PaaS solutions.
- Provide all the customers with their own copy of the database by using the PaaS solution.
- Provide the customers with different table and row access based on the customer's service agreement.
- In the event of an Azure regional outage, ensure that the customers can access the PaaS solution with minimal downtime. The solution must provide automatic failover.
- Ensure that users of the PaaS solution can create their own database objects but be prevented from modifying any of the existing database objects supplied by Contoso.

Technical Requirements
Contoso identifies the following technical requirements:
- Users of the PaaS solution must be able to sign in by using their own corporate Microsoft Entra credentials or have Microsoft Entra credentials supplied to them by Contoso. The solution must avoid using the internal Microsoft Entra ID of Contoso to minimize guest users.
- All customers must have their own resource group, Azure SQL server, and Azure SQL database. The deployment of resources for each customer must be done in a consistent fashion.
- Users must be able to review the queries issued against the PaaS databases and identify any new objects created.
- Downtime during the PostgreSQL database migration must be minimized.

Monitoring Requirements
Contoso identifies the following monitoring requirements:
- Notify administrators when a PaaS database has a higher than average CPU usage.
- Use a single dashboard to review security and audit data for all the PaaS databases.
- Use a single dashboard to monitor query performance and bottlenecks across all the PaaS databases.
- Monitor the PaaS databases to identify poorly performing queries and resolve query performance issues automatically whenever possible.

PaaS Prototype
During prototyping of the PaaS solution in Azure, you record the compute utilization of a customer's Azure SQL database as shown in the following exhibit.
Exhibit (q080-2.png): Azure portal "Compute utilization" chart, Show data for last: 7 days, Aggregation type: Max. Series: CPU percentage (Max), Data IO percentage (Max), Log IO percentage (Max). X-axis roughly Feb 4 - Feb 10 (UTC-06:00). Utilization is ~0% most of the time with short, intermittent spikes roughly once per day (spikes reaching 40-100%: Data IO up to ~100%, Log IO up to ~100% on some spikes, CPU typically 20-50%, one ~100%). Pattern = intermittent/bursty usage with long idle periods.

Role Assignments
For each customer's Azure SQL Database server, you plan to assign the roles shown in the following exhibit.
Exhibit (q080-3.png): Azure portal Access control > Role assignments, filtered Type: Groups, Role: 2 selected, Scope: All scopes; "Total number of role assignments: 15"; 2 items (2 Groups):
| Role | Name | Type | Role | Scope |
|---|---|---|---|---|
| Contributor | DBAGroup1 | Group | Contributor | This resource |
| SQL DB Contributor | DBAGroup2 | Group | SQL DB Contributor | This resource |

---

---

## Excluded questions (not in current outline)

These appeared in the sources but test skills that are not in the DP-300 skills outline as of 27 Oct 2026 (mostly legacy DP-200/DP-203 content or retired features), so they are not included above.

| Source IDs | Topic | Reason excluded |
|---|---|---|
| MD-214 | Migration/sync: Data Sync DB1 - BULK INSERT option + CREATE INDEX param | SQL Data Sync - not in current outline (retiring) |
| MD-064 | Migration/sync: Data Sync group with private link - approve private endpoint | SQL Data Sync - not in current outline (retiring) |
| MD-212 | Migration/sync: Data Sync without public endpoint - Private Link + snapshot isolation | SQL Data Sync - not in current outline (retiring) |
| MD-054 | Migration/sync: SQL Data Sync between SQL1 and SQL2 setup sequence | SQL Data Sync - not in current outline (retiring) |
| MD-110 | Classification: register DB1 in Purview for scan | Microsoft Purview governance - not in outline |
| MD-195 | Defender: centrally monitor SQL VMs + DBs for vulnerabilities | Microsoft Defender for SQL - removed from outline |
| MD-130 | Defender: SIM protect databases from SQL injection | Microsoft Defender for SQL - removed from outline |
| NOTE-235 | Governance: Azure Policy initiative for 20 policies on SQL DBs | Azure Policy - not in outline |
| MD-140 | Governance: SIM resource lock to prevent deleting db1 | Azure resource locks - not in outline |
| MD-207 | Monitoring: centralized for DBs/MIs/VMs - SQL Insights | Answer is retired SQL Insights |
| NOTE-030 | Table design: Employee data types to reduce storage | Data-type table design - not in outline |
| MD-092, CQ-072, NOTE-16 | ADLS Gen2: time-limited app access - SAS | Data Lake Storage |
| MD-097, CQ-077 | Data Factory: CMK encryption first step | Data Factory |
| CQ-020 | Data Factory: Git config adf_publish | Data Factory |
| NOTE-057 | Data Factory: storage event trigger - Microsoft.EventGrid | Data Factory |
| MD-168 | Databricks: diagnostic log for compute changes - clusters | Databricks |
| CQ-018 | Databricks: magic command %<language> | Databricks |
| CQ-009 | Databricks: partitionBy StoreID/Year/... parquet | Databricks |
| MD-007, CQ-024 | Databricks: reduce cluster start time - pool | Databricks |
| MD-164 | Databricks: send metrics to Azure Monitor sequence | Databricks |
| MD-163 | Databricks: Spark analytics for intrusion detection | Databricks |
| CQ-021, NOTE-10-img030 | Databricks: structured streaming output mode append | Databricks |
| CQ-010 | Event Hubs Dedicated: change partition count | Event Hubs |
| MD-198, NOTE-002 | SQL Insights: collect with / store in | retired SQL Insights |
| MD-191 | SQL Insights: two resources to create first | retired SQL Insights |
| CQ-002 | SSISDB after master loss - attach, TRUSTWORTHY, open/encrypt master key | SSIS catalog admin |
| NOTE-145 | Star schema: date dimension YYYYMMDD key | DW star-schema design |
| CQ-014, NOTE-21 | Star schema: website analytics columns to tables | DW star-schema design |
| MD-153 | Stream Analytics (ADatum v2): counter for sizing - SU% | Stream Analytics |
| MD-152 | Stream Analytics (ADatum v2): windowing - tumbling | Stream Analytics |
| CQ-017, NOTE-55 | Stream Analytics: anomaly detection spikes/dips | Stream Analytics |
| MD-166, NOTE-063 | Stream Analytics: backlogged input events - increase SUs | Stream Analytics |
| MD-169 | Stream Analytics: doubling devices - watermark delay | Stream Analytics |
| MD-162 | Stream Analytics: TopOne per game tumbling 5 min | Stream Analytics |
| NOTE-60, MD-167 | Stream Analytics: two metrics besides SU% | Stream Analytics |
| NOTE-258 | Synapse (unknown retail case): table types for retail store / promotional data | Synapse distribution |
| MD-010, CQ-027 | Synapse series file DateTime: dedicated pool external table | Synapse external tables |
| MD-008, CQ-025 | Synapse series file DateTime: Get Metadata activity | Synapse pipelines |
| MD-009, CQ-026 | Synapse series file DateTime: serverless external table | Synapse serverless |
| CQ-011, NOTE-img075 | Synapse serverless: address-only table from Parquet | Synapse serverless CETAS |
| CQ-022, NOTE-155 | Synapse serverless: OPENROWSET tripdata_2020* FIRSTROW | Synapse serverless |
| CQ-012 | Synapse Spark: load JSON varying structure - PySpark | Synapse Spark |
| CQ-016 | Synapse Spark: tables visible in serverless pool - Parquet | Synapse Spark |
| CQ-008 | Synapse: CREATE EXTERNAL DATA SOURCE ADLS - dfs + HADOOP | Synapse PolyBase |
| MD-165, NOTE-023 | Synapse: data loads memory - larger resource class | Synapse workload mgmt |
| MD-171, NOTE-005 | Synapse: data skew - dm_pdw_nodes_db_partition_stats | Synapse dedicated pool |
| CQ-013, NOTE-3-img043 | Synapse: date dimension distribution - REPLICATE | Synapse distribution |
| CQ-015, NOTE-009 | Synapse: FactSales distribution + partition column | Synapse distribution |
| MD-172 | Synapse: PDW_SHOWSPACEUSED skew | Synapse dedicated pool |
| CQ-019, NOTE-006 | Synapse: PolyBase load parquet sequence | Synapse PolyBase |
| MD-170 | Synapse: slow common queries - cache hit percentage | Synapse dedicated pool |
| NOTE-66 | Synapse: surrogate key for dimension table | Synapse DW design |
| NOTE-085 | Synapse: website-visits fact table distribution + index | Synapse distribution |
| MD-062 | Migration: what DMA migrates to Azure SQL DB | Core skill is trivia about the Data Migration Assistant (which objects DMA migrates). DMA was retired on 16 Jul 2025 and this isn't a current migration skill. Source MD-062 key B (functions, stored procedures, views, and roles) is correct for the classic exam. |
