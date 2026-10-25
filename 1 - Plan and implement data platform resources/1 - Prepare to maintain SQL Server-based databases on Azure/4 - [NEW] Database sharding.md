## 🧱 **What Is Sharding?**

- **Sharding** = **horizontal scale-out**. Rows of identically structured tables are spread across **many independent databases (shards)**.
- Each row belongs to a shard based on its **sharding key** (often `TenantID` or `CustomerID`).
- **Scale up (vertical)** = more vCores/storage for **one** database. **Scale out (horizontal)** = **more databases**.
- DP-300 skill: **"Recommend a database sharding solution"**. Expect questions about **when to shard** and **which strategy or tool to use**.

---

## 🤔 **Shard or Not? (Try simpler options first)**

| Requirement                                              | Best fit                                               |
| -------------------------------------------------------- | ------------------------------------------------------ |
| Database is outgrowing **4 TB** (GP/BC) storage           | **Hyperscale** (up to **128 TB**), no app changes      |
| Need more read capacity                                  | **Read scale-out** / Hyperscale **named replicas** (up to 30) |
| Many small databases with **unpredictable, varying load** | **Elastic pool** (shared resources, cheaper)           |
| Total throughput/data beyond **any single database**     | **Sharding**                                           |
| Tenants need **physical isolation** (security/compliance)| **Database-per-tenant** (a form of sharding)           |
| Data must live in **different geographies**              | **Sharding** by region                                 |

- Docs: sharding works best when **every transaction can be limited to one sharding key value**, so all transactions stay local to one database.
- Microsoft's own migration guidance says: if you sharded only because of **storage limits**, consider **consolidating into Hyperscale**.
- Sharding adds **app complexity**: routing, cross-shard queries, rebalancing and schema changes across N databases. Recommend it only when the requirement really needs it.

---

## 🔑 **Choosing a Shard Key**

- It must appear in **every sharded table**, ideally as the **leading column of the primary key / unique index**. Split-merge needs this.
- **High cardinality** + **even distribution** → no hot shards.
- Matches the **most common access pattern**, so most queries hit **one** shard.
- **Stable**: changing a row's key means moving it between shards.
- Supported key types (Elastic Database client library): **int, bigint (long), GUID (uniqueidentifier), varbinary (byte[]), datetime, timespan, datetimeoffset**.

---

## 🧭 **Sharding Strategies**

### 1. **Range-based sharding**

- Data is split by **value ranges** (e.g., IDs 1–1M → Shard A, 1M–2M → Shard B).
- ✅ Easy to understand, and range queries stay local.
- ❌ Risk of **hot spots**: if new data always lands in the newest range, that shard gets overloaded.

### 2. **Hash-based sharding**

- A **hash function** on the shard key picks the shard (e.g., `UserID % 4 → Shard #`).
- ✅ Spreads load evenly and avoids hot spots.
- ❌ **Range queries** (e.g., "all users between 1000 and 2000") become **hard/expensive**, because the data is scattered.
- ⚠️ The Elastic Database client library only offers **list** and **range** shard maps. For hash sharding, the app computes a hash bucket and maps the buckets with a list/range map.

### 3. **Lookup (Directory) sharding**

- A **mapping table (directory)** says where each key lives. The **shard map manager** is this pattern.
- ✅ Very flexible: you can move a single tenant without re-hashing everything.
- ❌ Extra lookup step adds overhead, and the directory must stay consistent and available.

👉 **DP-300 exam lens:**

- Range = good for sequential/natural splits, but load can be uneven.
- Hash = good for uniform distribution, bad for range queries.
- Lookup = flexible, but adds indirection.

---

## 🏢 **Multi-Tenant Patterns (SaaS)**

| Pattern                              | Isolation | Cost per tenant              | Scale                 | Notes                                              |
| ------------------------------------ | --------- | ---------------------------- | --------------------- | -------------------------------------------------- |
| **Standalone app + single DB**       | Highest   | Highest (sized for peaks)    | 1–100s tenants        | Separate RG/subscription per tenant, so no shared pool |
| **Database-per-tenant**              | High      | Low with **elastic pools**   | 1–100,000s            | Per-tenant restore and schema customization are easy |
| **Sharded multi-tenant DBs**         | Low       | **Lowest** for small tenants | 1–1,000,000s          | Needs tenant ID in schema + **row-level security** |
| **Hybrid**                           | Mixed     | Mixed                        | Very high             | Free-trial tenants shared; premium tenants get own DB |

- **Database-per-tenant** = **list mapping** (one key → one database).
- **Multi-tenant shards** = **range mapping** (a range of tenants → one database), or a list mapping with several keys per database.
- Put shards in an **elastic pool** for cost efficiency.
- ⚠️ You **can't restore a multi-tenant DB to a single tenant**. Use split-merge (or custom ETL) to move a tenant out.

---

## 🧰 **Elastic Database Tools (Azure SQL Database only)**

| Tool                                        | Purpose                                                                                     | Status (Oct 2026)                                    |
| ------------------------------------------- | ------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| **Elastic Database client library** (.NET/Java) | Creates and manages the **shard map**; does **data-dependent routing** (`OpenConnectionForKey`) and multi-shard queries | Supported (not available in SQL database in Fabric) |
| **Shard map manager (GSM + LSM)**           | **Global Shard Map** DB holds all mappings; each shard has a **Local Shard Map** (`__ShardManagement` schema) | Part of the client library                           |
| **Split-merge tool**                        | Moves **shardlets** between shards (split/merge/move) and keeps the shard map in sync  | ⚠️ Self-hosted **Azure Cloud Services** package (last NuGet release 2015). Cloud Services (classic) was **retired Sept 2024**. Treat it as legacy |
| **Elastic query** (`RDBMS` = vertical)      | Read-only cross-database T-SQL via external tables                                         | **Still preview**                                    |
| **Elastic query** (`SHARD_MAP_MANAGER` = horizontal) | Fan-out reporting across all shards                                                | **End of support 31 Mar 2027**; no new data sources after that |
| **Elastic jobs**                            | Run T-SQL across servers/pools/databases on a schedule                                     | **GA**                                               |
| **Elastic transactions**                    | Distributed transactions across Azure SQL DBs (.NET `System.Transactions`)                 | Supported                                            |

- **Shardlet** = all the data for **one sharding key value** (e.g., one tenant).
- The **shard map manager** methods change **metadata only**. They never move data. Data movement = split-merge or your own process.
- Apps that only **route** queries need **read-only** access to the GSM. Admin tools that **change** maps need read/write on the GSM **and** every shard.

---

## 🔎 **Querying and Managing Across Shards**

```sql
-- Elastic query (horizontal) - run on the "head" database
CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<strong password>';
CREATE DATABASE SCOPED CREDENTIAL ShardCred
    WITH IDENTITY = 'shard_reader', SECRET = '<password>';

CREATE EXTERNAL DATA SOURCE ShardSrc WITH (
    TYPE = SHARD_MAP_MANAGER,                       -- end of support 31-Mar-2027
    LOCATION = 'myserver.database.windows.net',
    DATABASE_NAME = 'ShardMapManagerDb',
    CREDENTIAL = ShardCred,
    SHARD_MAP_NAME = 'TenantShardMap');

CREATE EXTERNAL TABLE dbo.Orders (TenantId int, OrderId int, Amount money)
WITH (DATA_SOURCE = ShardSrc, DISTRIBUTION = SHARDED(TenantId));

SELECT TenantId, SUM(Amount) FROM dbo.Orders GROUP BY TenantId;   -- fans out to all shards
```

- Elastic query: **read-only**, **SQL authentication only**, no private link to target DBs, and **one unavailable shard fails the whole query**.
- **Elastic jobs** for schema changes and maintenance across all shards:
  - Target groups can be a **server**, **elastic pool**, or **database**. Servers and pools are **enumerated dynamically at run time**, so new shards are picked up automatically.
  - Auth: **user-assigned managed identity** (recommended) or database-scoped credentials. **System-assigned MI isn't supported.**
  - Job database: **S1 or higher** recommended; **can't be Hyperscale**. Agent tiers JA100–JA800 set concurrent targets. Scripts must be **idempotent**.
- Alternatives to elastic query fan-out: **mirroring to Microsoft Fabric**, Azure Data Factory/ETL, **Hyperscale** consolidation, elastic jobs (no aggregation), or **SQL Managed Instance** (native cross-database / linked server queries).

---

## ✅ **Summary**

- Shard **only** when a single database (even **Hyperscale** or an **elastic pool**) can't meet scale, isolation or geography needs.
- Pick a **high-cardinality, evenly distributed, query-aligned** shard key. Make it the leading PK column.
- **Range** = simple but hot spots; **hash** = even load but bad for ranges; **lookup/directory** = flexible but adds an extra lookup.
- **Database-per-tenant + elastic pool** = strong isolation at low cost. **Sharded multi-tenant + RLS** = lowest cost at massive tenant counts.
- Tooling: **client library + shard map manager** for routing, **elastic jobs** for fleet management, **elastic query** for reporting (preview; SMM mode ends Mar 2027).

---

## 🎯 **Exam Tips / Traps**

- "DB will grow to 50 TB, **minimize app changes**" → **Hyperscale**, not sharding.
- "Thousands of tenants, each needs **isolation** and **per-tenant restore**, lowest cost" → **database-per-tenant in elastic pools**.
- "Millions of tiny tenants, lowest cost" → **sharded multi-tenant databases** + **row-level security**.
- "Route a connection to the right shard by key" → **Elastic Database client library** (data-dependent routing).
- "Run the same schema change on every shard, including shards added later" → **elastic job** targeting the **server/pool** (dynamic enumeration).
- "Rebalance a hot tenant to a new shard" → **split-merge** (move shardlet), which updates the shard map.
- "Report across all shards with T-SQL" → elastic query (preview, **read-only**). Know that **SHARD_MAP_MANAGER mode ends 31 Mar 2027**.
- These tools are **Azure SQL Database** features. On **SQL Managed Instance** use 3-/4-part names or linked servers instead.

> Source: https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-scale-introduction · https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-scale-shard-map-management · https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-scale-overview-split-and-merge · https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-query-overview · https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-query-horizontal-partitioning-migration · https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-overview · https://learn.microsoft.com/en-us/azure/azure-sql/database/saas-tenancy-app-design-patterns · https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tier-hyperscale · https://learn.microsoft.com/en-us/previous-versions/azure/cloud-services/cloud-services-choose-me
