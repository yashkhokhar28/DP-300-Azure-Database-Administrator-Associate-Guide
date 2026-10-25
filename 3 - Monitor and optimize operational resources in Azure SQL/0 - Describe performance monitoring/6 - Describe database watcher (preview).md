### ✅ **Database Watcher in Azure SQL — Explained (Preview Feature)**

---

### 📊 **What Is Database Watcher?**

**Database Watcher** is a **new Azure monitoring solution (currently in preview)** designed specifically for:

- **Azure SQL Database**
- **Azure SQL Managed Instance**

It provides:

- **Centralized, near real-time performance monitoring**
- Health insights for your SQL resources
- Customizable dashboards and alerting

---

### 🔍 **Key Features**

| Feature                        | Description                                                                                                                 |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| **Central data collection**    | Collects monitoring data from SQL Databases, Elastic Pools, and Managed Instances.                                          |
| **Fast data ingestion**        | Monitoring data is collected and ingested within **seconds**, allowing for low-latency analysis.                            |
| **Customizable targets**       | You can configure **which SQL resources to monitor**, how often to collect data, and for how long to retain it.             |
| **Dashboards in Azure Portal** | Provides a **single pane of glass** with dashboards showing health and performance insights.                                |
| **Alerting**                   | Includes **pre-built alert templates** and lets you create custom alerts to notify on critical events.                      |
| **Data storage flexibility**   | Lets you store monitoring data in: <br>• **Azure Data Explorer (ADX)** <br>• **Real-Time Intelligence (RTI) in Microsoft Fabric** (formerly Real-Time Analytics) 🔄 |

---

### 🔧 **How Database Watcher Works**

- **Data flow diagram:**

  - SQL Database / Managed Instance ➡ **Database Watcher** ➡ **Data Store (ADX or Fabric RTI)** ➡ **Azure Portal Dashboards & Analytics Tools** 🔄

- **Supported data analysis tools:**

  - Azure Data Explorer (Kusto Query Language)
  - Real-Time Intelligence in Microsoft Fabric (formerly Real-Time Analytics) 🔄
  - Other Azure Monitor tools

- **Latency:** Monitoring data is collected and available **within seconds**, enabling fast troubleshooting and insights.

---

### 🛡️ **What Can You Monitor?**

- SQL performance counters (CPU, I/O, waits, etc.)
- Query performance
- Deadlocks and blocking sessions
- Elastic pool usage
- Database configuration changes
- Managed Instance health metrics

---

### 🚨 **Alerts and Templates**

- Pre-built templates for **common alerts**.
- Allows **custom alert creation** for your specific needs.

  - Example: Alert when DTU usage exceeds 80%, or when a deadlock is detected.

---

### 🏗️ **Steps to Configure Database Watcher**

1. **In Azure Portal**, search for **“Database Watchers”**, then click **Create**.
2. Fill in:

   - **Subscription & Resource Group**
   - **Watcher Name**
   - **Azure Region**

3. **Configure the Data Store:**

   - Use Azure Data Explorer (full or **free** cluster) or a **Real-Time Intelligence** database in Microsoft Fabric as your storage for monitoring data. 🔄

4. **Add SQL Targets:**

   - Select **Azure SQL Databases** or **Managed Instances** to monitor.

5. Review your configuration and click **Create**.

---

### 🛠️ **Data Storage Options**

| Data Store Type                                | Usage                                          |
| ---------------------------------------------- | ---------------------------------------------- |
| **Azure Data Explorer (ADX)**                  | Recommended for high-volume, time-series data. |
| **Free ADX Cluster**                           | For small workloads or testing.                |
| **Real-Time Intelligence (RTI) in Microsoft Fabric** | For low-latency, real-time analysis (SaaS, same engine as ADX). 🔄 |

---

### ⚠️ **Limitations to Note (Preview Status)**

- Supports **all** SQL DB / MI service tiers & purchasing models (DTU, vCore, serverless, Hyperscale, elastic pools) and readable/geo/named replicas — but **not SQL Server on Azure VMs**. 🔄
- Data retention periods and query capabilities may evolve in future releases.
- Limits (preview): **100 SQL targets per watcher**, **20 watchers per subscription**. 🔄
- **Alerts** (Azure Monitor log search alerts, added Feb 2025) aren't available when the data store is **Fabric RTI** or a **free ADX cluster**. 🔄
- Cost: watchers & dashboards are **free**; you pay for the ADX cluster / Fabric capacity (plus alerts, and Key Vault if using SQL auth). A new watcher isn't started automatically → select **Start**. 🔄

---

### ✔️ **Summary Takeaways**

- Database Watcher is a **powerful, cloud-native SQL monitoring tool**.
- Helps you get **near real-time performance and health insights** across your Azure SQL environment.
- Integrates easily with Azure analytics platforms for advanced querying and visualization.
- Still in **preview**, but promising for proactive SQL monitoring and alerting.

---
