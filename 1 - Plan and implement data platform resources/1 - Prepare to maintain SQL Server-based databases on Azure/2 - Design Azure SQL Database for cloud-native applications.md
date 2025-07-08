## **Azure SQL Database Overview**

Azure SQL Database is a **Platform as a Service (PaaS)** offering, providing **scalable, low-maintenance SQL databases** ideal for modern cloud applications.

---

## **Purchasing Models**

Two models available:

1. **vCore-based:** Separate scaling for compute and storage. Service tiers:

   - **General Purpose:** Budget-friendly, balanced resources.
   - **Business Critical:** High performance, In-Memory OLTP, SSD storage.
   - **Hyperscale:** Large-scale workloads with automatic scaling.

2. **DTU-based:** Fixed compute & storage bundles (Basic, Standard, Premium). Scale up DTUs when storage or compute needs increase.

   > ❗ DTU model **not supported** on SQL Managed Instance.

---

## **Serverless Tier**

- **Auto-scales** compute based on usage.
- Can **auto-pause** when idle to save costs.
- Pay only for storage when paused.
- Great for development/test environments.

---

## **Deployment Models**

- **Single Database:** Individually managed databases.
- **Elastic Pool:** Share resources among multiple databases (cost-effective for apps with many small databases).
- Manage resources using Azure Portal, CLI, ARM templates, or PowerShell.

---

## **Network Access**

- Databases accessible through public endpoints or securely via **Virtual Network** and **Private Link**.
- Access controlled through firewall rules.

---

## **Backup & Restore Features**

- **Automatic Backups:** Full, differential, and transaction log backups.
- **Geo-restore:** Restore to another region.
- **Point-in-Time Restore (PITR):** Recover data within the last 1–35 days.
- **Long-Term Retention (LTR):** Keep backups up to 10 years.

---

## **Automatic Tuning**

Uses **machine learning** to optimize performance:

- Adds/removes indexes.
- Forces good execution plans.
- Identifies expensive queries.

---

## **Elastic Query (Preview)**

Run **T-SQL queries across multiple databases**, supporting:

- **Vertical partitioning:** Different schemas.
- **Horizontal partitioning (sharding):** Same schema, split data.

---

## **Elastic Jobs**

Cloud replacement for SQL Server Agent:

- Automates maintenance across multiple databases, subscriptions, and regions.
- Executes tasks in parallel.
- **Not available in SQL Managed Instance.**

---

## **SQL Database in Microsoft Fabric**

Integrates SQL databases with **Microsoft Fabric**, a unified platform for data analysis and application building:

- Supports both **operational and analytical workloads**.
- Enables collaborative workflows for data engineers and analysts.

---

### ✅ **Summary Takeaway:**

Azure SQL Database is a **scalable, cloud-native SQL platform** with flexible pricing, advanced scaling, backup, security, and performance tuning—ideal for both new cloud apps and cost-effective multi-database SaaS models.

---
