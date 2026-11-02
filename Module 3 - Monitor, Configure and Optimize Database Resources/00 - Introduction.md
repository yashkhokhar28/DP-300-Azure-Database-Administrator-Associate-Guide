# Module 3: Monitor, Configure and Optimize Database Resources

**Exam weight:** 20–25% · **Reading time:** ~96 min · **Files:** 10

## What you will learn
You will learn how to tell when a database is slow, and why. You will learn the monitoring tools for VMs, Azure SQL Database and Managed Instance. You will learn to set compute, storage and server settings and keep indexes healthy. You will learn to read plans, use Query Store, fix blocking and rewrite slow queries.

Think of a busy restaurant. The manager writes down a normal evening (the **baseline**) and watches the kitchen with cameras (**monitoring tools**). They pick enough chefs and a big enough fridge (**compute and storage**) and clean every night (**maintenance**). Each recipe card shows every step (**execution plan**), and a logbook remembers which recipe worked best (**Query Store**). Two chefs fighting over one stove make a queue (**blocking**). A tidy pantry makes every dish faster (**indexes**).

## Lessons in this module
| # | Lesson | What it covers | Time |
|---|--------|----------------|------|
| 1 | [Performance baselines and monitoring tools](<01 - Performance baselines and monitoring tools.md>) | Baselines, metric sources, alerts, QPI | ~11 min |
| 2 | [Extended Events, database watcher and Intelligent Insights](<02 - Extended Events, database watcher and Intelligent Insights.md>) | XE sessions, database watcher, SQLInsights | ~11 min |
| 3 | [Configure compute, storage and server settings](<03 - Configure compute, storage and server settings.md>) | Disks, VM size, tempdb, scaling, MAXDOP, Resource Governor | ~12 min |
| 4 | [Database maintenance](<04 - Database maintenance.md>) | Indexes, statistics, DBCC CHECKDB, scoped config | ~10 min |
| 5 | [Automatic tuning and intelligent query processing](<05 - Automatic tuning and intelligent query processing.md>) | Plan correction, auto indexes, IQP | ~9 min |
| 6 | [Execution plans and DMVs](<06 - Execution plans and DMVs.md>) | Estimated vs actual plans, key DMVs | ~9 min |
| 7 | [Query Store](<07 - Query Store.md>) | Settings, reports, plan forcing, hints | ~8 min |
| 8 | [Blocking, locking and wait statistics](<08 - Blocking, locking and wait statistics.md>) | Locks, deadlocks, RCSI, wait types | ~11 min |
| 9 | [Index design and query tuning](<09 - Index design and query tuning.md>) | Index types, design, query rewrites, hints | ~12 min |

## Words you will meet
| Word | Simple meaning |
|------|----------------|
| Baseline | Your normal performance numbers, recorded to compare against later |
| DMV | A built-in system view (`sys.dm_...`) that shows what SQL Server is doing now |
| Execution plan | The step-by-step recipe SQL Server uses to run a query |
| Query Store | A recorder in the database that keeps query plans and run times over time |
| Plan regression | A new plan that is slower than an older one |
| Statistics | Notes on how values are spread in a column, used to guess row counts |
| MAXDOP | The most CPU cores one query can use |
| Resource Governor | A feature that limits CPU, memory and I/O per group of users |
| Blocking | One session waiting for another to release a lock |
| SARGable | Written so SQL Server can use an index seek |

## Exam skills covered
- [ ] Prepare an operational performance baseline → 1
- [ ] Determine sources for performance metrics → 1
- [ ] Interpret performance metrics → 1, 8
- [ ] Configure and monitor activity and performance → 1
- [ ] Monitor by using database watcher → 2
- [ ] Monitor by using Extended Events → 2
- [ ] Configure Query Store → 7
- [ ] Monitor by using Query Store → 7
- [ ] Identify and resolve session blocking → 8
- [ ] Identify performance issues using dynamic management views (DMVs) → 6, 8
- [ ] Identify and implement index changes for queries → 9
- [ ] Recommend query construct modifications based on resource usage → 9
- [ ] Review execution plans → 6
- [ ] Monitor by using Intelligent Insights → 2
- [ ] Implement index maintenance tasks → 4
- [ ] Implement statistics maintenance tasks → 4
- [ ] Implement database integrity checks → 4
- [ ] Configure database automatic tuning → 5
- [ ] Configure server settings for performance → 3
- [ ] Configure Resource Governor for performance → 3
- [ ] Implement database-scoped configuration → 4
- [ ] Configure compute and storage resources for scaling → 3
- [ ] Identify use cases for intelligent query processing (IQP) features → 5
