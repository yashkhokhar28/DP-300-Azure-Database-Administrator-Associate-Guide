# DP-300 Syllabus Tracker

Every skill from the **official DP-300 skills outline (as of 27 October 2026)**, mapped to the notes in this repo.
Tick a box (`- [x]`) once you've studied that skill and can explain it without looking.

- Source: [Study guide for Exam DP-300](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/dp-300)
- The only change from the previous outline: *"Always Encrypted with VBS enclaves"* became *"Always Encrypted with **secure** enclaves"*.
- 🆕 = note added in Oct 2026 to fill a gap in the original course notes.
- 🔄 at the end of a line = corrected/added during the Oct 2026 verification against Microsoft docs — see [VERIFICATION-LOG.md](VERIFICATION-LOG.md).

**Final-week revision:** the condensed summaries in [`revision/`](revision), the IaaS-vs-PaaS comparisons in [`Diffrences.md`](Diffrences.md), the [DMV playbook](<Azure SQL DMVs list.md>), practice questions in [`Questions.md`](Questions.md), and the [official practice assessment](https://learn.microsoft.com/en-us/credentials/certifications/exams/dp-300/practice/assessment?assessment-type=practice&assessmentId=58).


---

## 1. Plan and implement data platform resources (15–20%)


### Plan and deploy Azure SQL solutions

- [ ] **Recommend a database offering based on specific requirements**
  - [Understand SQL Server in an Azure virtual machine](<1 - Plan and implement data platform resources/1 - Prepare to maintain SQL Server-based databases on Azure/1 - Understand SQL Server in an Azure virtual machine.md>) · [Design Azure SQL Database for cloud-native applications](<1 - Plan and implement data platform resources/1 - Prepare to maintain SQL Server-based databases on Azure/2 - Design Azure SQL Database for cloud-native applications.md>) · [Explain IaaS options to deploy SQL Server in Azure](<1 - Plan and implement data platform resources/2 - Deploy IaaS solutions with Azure SQL/2 - Explain IaaS options to deploy SQL Server in Azure.md>) · [Explain PaaS options for deploying SQL Server in Azure](<1 - Plan and implement data platform resources/3 - Deploy PaaS solutions with Azure SQL/1 - Explain PaaS options for deploying SQL Server in Azure.md>) · [Choose the right Azure SQL Database feature](<1 - Plan and implement data platform resources/4 - Migrate SQL Server workloads to Azure SQL Database/2 - Choose the right Azure SQL Database feature.md>) · [Diffrences](<Diffrences.md>)
- [ ] **Choose an automated deployment method**
  - [Describe deployment models in Azure](<5 - Automate database tasks for Azure SQL/1 - Automate database deployment/2 - Describe deployment models in Azure.md>) · [Automate deployment by using Azure Resource Manager templates and Bicep](<5 - Automate database tasks for Azure SQL/1 - Automate database deployment/3 - Automate deployment by using Azure Resource Manager templates and Bicep.md>) · [Automate deployment by using PowerShell](<5 - Automate database tasks for Azure SQL/1 - Automate database deployment/4 - Automate deployment by using PowerShell.md>) · [Automate deployment by using Azure CLI](<5 - Automate database tasks for Azure SQL/1 - Automate database deployment/5 - Automate deployment by using Azure CLI.md>)
- [ ] **Identify use cases for Azure Arc-enabled SQL services**
  - [Understand hybrid scenarios](<1 - Plan and implement data platform resources/2 - Deploy IaaS solutions with Azure SQL/3 - Understand hybrid scenarios.md>) · [Understand SQL Server in an Azure virtual machine](<1 - Plan and implement data platform resources/1 - Prepare to maintain SQL Server-based databases on Azure/1 - Understand SQL Server in an Azure virtual machine.md>)
- [ ] **Identify use cases for Azure SQL Database in Microsoft Fabric**
  - [SQL database in Microsoft Fabric 🆕](<1 - Plan and implement data platform resources/1 - Prepare to maintain SQL Server-based databases on Azure/5 - [NEW] SQL database in Microsoft Fabric.md>) · [Design Azure SQL Database for cloud-native applications](<1 - Plan and implement data platform resources/1 - Prepare to maintain SQL Server-based databases on Azure/2 - Design Azure SQL Database for cloud-native applications.md>)
- [ ] **Plan for table partitioning**
  - [Table partitioning and data compression 🆕](<1 - Plan and implement data platform resources/2 - Deploy IaaS solutions with Azure SQL/6 - [NEW] Table partitioning and data compression.md>) · [Explore performance and security](<1 - Plan and implement data platform resources/2 - Deploy IaaS solutions with Azure SQL/4 - Explore performance and security.md>)
- [ ] **Recommend a database sharding solution**
  - [Database sharding 🆕](<1 - Plan and implement data platform resources/1 - Prepare to maintain SQL Server-based databases on Azure/4 - [NEW] Database sharding.md>) · [Choose the right Azure SQL Database feature](<1 - Plan and implement data platform resources/4 - Migrate SQL Server workloads to Azure SQL Database/2 - Choose the right Azure SQL Database feature.md>) · [temp](<temp.md>)
- [ ] **Deploy database offerings on selected platforms**
  - [Explain IaaS options to deploy SQL Server in Azure](<1 - Plan and implement data platform resources/2 - Deploy IaaS solutions with Azure SQL/2 - Explain IaaS options to deploy SQL Server in Azure.md>) · [Explore single SQL database](<1 - Plan and implement data platform resources/3 - Deploy PaaS solutions with Azure SQL/2 - Explore single SQL database.md>) · [Deploy SQL database elastic pool](<1 - Plan and implement data platform resources/3 - Deploy PaaS solutions with Azure SQL/3 - Deploy SQL database elastic pool.md>) · [Examine SQL managed instance](<1 - Plan and implement data platform resources/3 - Deploy PaaS solutions with Azure SQL/5 - Examine SQL managed instance.md>)
- [ ] **Deploy hybrid SQL Server solutions**
  - [Understand hybrid scenarios](<1 - Plan and implement data platform resources/2 - Deploy IaaS solutions with Azure SQL/3 - Understand hybrid scenarios.md>) · [Describe hybrid solutions](<6 - Plan and implement a high availability and disaster recovery environment/1 - Describe high availability and disaster recovery strategies/7 - Describe hybrid solutions.md>)
- [ ] **Apply patches and updates for hybrid and IaaS deployments**
  - [Patching hybrid and IaaS SQL Server 🆕](<1 - Plan and implement data platform resources/2 - Deploy IaaS solutions with Azure SQL/7 - [NEW] Patching hybrid and IaaS SQL Server.md>) · [Explain IaaS options to deploy SQL Server in Azure](<1 - Plan and implement data platform resources/2 - Deploy IaaS solutions with Azure SQL/2 - Explain IaaS options to deploy SQL Server in Azure.md>)

### Configure resources for scale and performance

- [ ] **Configure Azure SQL Database for scale and performance**
  - [Explore single SQL database](<1 - Plan and implement data platform resources/3 - Deploy PaaS solutions with Azure SQL/2 - Explore single SQL database.md>) · [Deploy SQL database elastic pool](<1 - Plan and implement data platform resources/3 - Deploy PaaS solutions with Azure SQL/3 - Deploy SQL database elastic pool.md>) · [Understand SQL database hyperscale](<1 - Plan and implement data platform resources/3 - Deploy PaaS solutions with Azure SQL/4 - Understand SQL database hyperscale.md>) · [Optimize database storage](<3 - Monitor and optimize operational resources in Azure SQL/1 - Configure SQL Server resources for optimal performance/4 - Optimize database storage.md>)
- [ ] **Configure Azure SQL Managed Instance for scale and performance**
  - [Examine SQL managed instance](<1 - Plan and implement data platform resources/3 - Deploy PaaS solutions with Azure SQL/5 - Examine SQL managed instance.md>) · [Optimize database storage](<3 - Monitor and optimize operational resources in Azure SQL/1 - Configure SQL Server resources for optimal performance/4 - Optimize database storage.md>)
- [ ] **Configure SQL Server on Azure Virtual Machines for scale and performance**
  - [Explore performance and security](<1 - Plan and implement data platform resources/2 - Deploy IaaS solutions with Azure SQL/4 - Explore performance and security.md>) · [Explain how to optimize Azure storage for SQL Server virtual machines](<3 - Monitor and optimize operational resources in Azure SQL/1 - Configure SQL Server resources for optimal performance/2 - Explain how to optimize Azure storage for SQL Server virtual machines.MD>) · [Describe virtual machine resizing](<3 - Monitor and optimize operational resources in Azure SQL/1 - Configure SQL Server resources for optimal performance/3 - Describe virtual machine resizing.md>)
- [ ] **Configure table partitioning**
  - [Table partitioning and data compression 🆕](<1 - Plan and implement data platform resources/2 - Deploy IaaS solutions with Azure SQL/6 - [NEW] Table partitioning and data compression.md>)
- [ ] **Configure data compression**
  - [Table partitioning and data compression 🆕](<1 - Plan and implement data platform resources/2 - Deploy IaaS solutions with Azure SQL/6 - [NEW] Table partitioning and data compression.md>) · [Explore performance and security](<1 - Plan and implement data platform resources/2 - Deploy IaaS solutions with Azure SQL/4 - Explore performance and security.md>)

### Plan and implement a migration strategy

- [ ] **Evaluate requirements for a migration**
  - [Migrate using Azure Arc 🆕](<1 - Plan and implement data platform resources/5 - Migrate SQL Server workloads to Azure SQL Managed Instance/7 - [NEW] Migrate using Azure Arc.md>) · [Examine data platform modernization](<1 - Plan and implement data platform resources/Migrate SQL Server workloads to Azure SQL/2 - Examine data platform modernization.md>) · [Plan a data migration](<1 - Plan and implement data platform resources/Migrate SQL Server workloads to Azure SQL/3 - Plan a data migration.md>) · [Understand Data Migration Assistant](<1 - Plan and implement data platform resources/Migrate SQL Server workloads to Azure Virtual Machine/7 -Understand Data Migration Assistant.md>) · [Choose the right Azure SQL Database feature](<1 - Plan and implement data platform resources/4 - Migrate SQL Server workloads to Azure SQL Database/2 - Choose the right Azure SQL Database feature.md>)
- [ ] **Evaluate offline or online migration strategies**
  - [Explore the available data migration technologies](<1 - Plan and implement data platform resources/Migrate SQL Server workloads to Azure SQL/4 - Explore the available data migration technologies.md>) · [Review migrations strategies and tools](<1 - Plan and implement data platform resources/Migrate SQL Server workloads to Azure Virtual Machine/2 - Review migrations strategies and tools.md>) · [Compare additional migration methods](<1 - Plan and implement data platform resources/Migrate SQL Server workloads to Azure Virtual Machine/4 -Compare additional migration methods.md>) · [Evaluate migration scenarios](<1 - Plan and implement data platform resources/5 - Migrate SQL Server workloads to Azure SQL Managed Instance/2 - Evaluate migration scenarios.md>)
- [ ] **Implement an online migration strategy**
  - [Use an online method to migrate to Azure SQL Database](<1 - Plan and implement data platform resources/4 - Migrate SQL Server workloads to Azure SQL Database/6 - Use an online method to migrate to Azure SQL Database.md>) · [Use Log Replay Service (LRS) to migrate](<1 - Plan and implement data platform resources/5 - Migrate SQL Server workloads to Azure SQL Managed Instance/3 - Use Log Replay Service (LRS) to migrate.md>) · [Migrate using Managed Instance link](<1 - Plan and implement data platform resources/5 - Migrate SQL Server workloads to Azure SQL Managed Instance/4 - Migrate using Managed Instance link.md>) · [Migrate using Azure Arc 🆕](<1 - Plan and implement data platform resources/5 - Migrate SQL Server workloads to Azure SQL Managed Instance/7 - [NEW] Migrate using Azure Arc.md>) · [Understand distributed availability group for migration](<1 - Plan and implement data platform resources/Migrate SQL Server workloads to Azure Virtual Machine/8 -Understand distributed availability group for migration.md>)
- [ ] **Implement an offline migration strategy**
  - [Use Azure Database Migration Service to migrate to Azure SQL Database 🆕](<1 - Plan and implement data platform resources/4 - Migrate SQL Server workloads to Azure SQL Database/8 - [NEW] Use Azure Database Migration Service to migrate to Azure SQL Database.md>) · [Migrate to Azure SQL Database using BACPAC](<1 - Plan and implement data platform resources/4 - Migrate SQL Server workloads to Azure SQL Database/5 - Migrate to Azure SQL Database using BACPAC.md>) · [Move data to Azure SQL Database](<1 - Plan and implement data platform resources/4 - Migrate SQL Server workloads to Azure SQL Database/7 - Move data to Azure SQL Database.md>) · [Migrate databases using native backup and restore](<1 - Plan and implement data platform resources/Migrate SQL Server workloads to Azure Virtual Machine/6 -Migrate databases using native backup and restore.md>)
- [ ] **Implement a migration to Azure**
  - [Use Azure Database Migration Service to migrate to Azure SQL Database 🆕](<1 - Plan and implement data platform resources/4 - Migrate SQL Server workloads to Azure SQL Database/8 - [NEW] Use Azure Database Migration Service to migrate to Azure SQL Database.md>) · [Migrate using Azure Arc 🆕](<1 - Plan and implement data platform resources/5 - Migrate SQL Server workloads to Azure SQL Managed Instance/7 - [NEW] Migrate using Azure Arc.md>) · [Use Azure SQL migration extension to migrate to Azure SQL Database](<1 - Plan and implement data platform resources/4 - Migrate SQL Server workloads to Azure SQL Database/3 - Use Azure SQL migration extension to migrate to Azure SQL Database.md>) · [Explore Data Migration Assistant to migrate to Azure SQL Database](<1 - Plan and implement data platform resources/4 - Migrate SQL Server workloads to Azure SQL Database/4 - Explore Data Migration Assistant to migrate to Azure SQL Database.md>) · [Explore Azure SQL Migration extension for Azure Data Studio](<1 - Plan and implement data platform resources/Migrate SQL Server workloads to Azure Virtual Machine/3 - Explore Azure SQL Migration extension for Azure Data Studio.md>) · [Explore Azure SQL Migration extension for Azure Data Studio](<1 - Plan and implement data platform resources/Migrate SQL Server workloads to Azure Virtual Machine/5 -Explore Azure SQL Migration extension for Azure Data Studio.md>)
- [ ] **Implement a migration between Azure SQL services**
  - [Migrate to Azure SQL Database using BACPAC](<1 - Plan and implement data platform resources/4 - Migrate SQL Server workloads to Azure SQL Database/5 - Migrate to Azure SQL Database using BACPAC.md>) · [Move data to Azure SQL Database](<1 - Plan and implement data platform resources/4 - Migrate SQL Server workloads to Azure SQL Database/7 - Move data to Azure SQL Database.md>) · [Move data to SQL Managed Instance](<1 - Plan and implement data platform resources/5 - Migrate SQL Server workloads to Azure SQL Managed Instance/5 - Move data to SQL Managed Instance.md>)
- [ ] **Implement Azure SQL Managed Instance database copy and move**
  - [Managed Instance database copy and move 🆕](<1 - Plan and implement data platform resources/5 - Migrate SQL Server workloads to Azure SQL Managed Instance/6 - [NEW] Managed Instance database copy and move.md>)
- [ ] **Troubleshoot a migration**
  - [Troubleshoot a migration 🆕](<1 - Plan and implement data platform resources/Migrate SQL Server workloads to Azure SQL/6 - [NEW] Troubleshoot a migration.md>) · [summary](<1 - Plan and implement data platform resources/Migrate SQL Server workloads to Azure SQL/5 - summary.md>)

---

## 2. Implement a secure environment (20–25%)


### Configure database authentication and authorization

- [ ] **Configure Microsoft Entra ID authentication for Azure SQL Database, Azure SQL Managed Instance, and SQL Server**
  - [Describe authentication and identities](<2 - Implement a secure environment for a database service/1 - Configure database authentication and authorization/2 - Describe authentication and identities.md>)
- [ ] **Configure authentication for SQL on Azure VMs and Azure SQL Managed Instance**
  - [Describe authentication and identities](<2 - Implement a secure environment for a database service/1 - Configure database authentication and authorization/2 - Describe authentication and identities.md>)
- [ ] **Configure security principals**
  - [Describe Security Principals](<2 - Implement a secure environment for a database service/1 - Configure database authentication and authorization/3 - Describe Security Principals.md>)
- [ ] **Create users from Microsoft Entra identities**
  - [Describe Security Principals](<2 - Implement a secure environment for a database service/1 - Configure database authentication and authorization/3 - Describe Security Principals.md>)
- [ ] **Configure database and object-level permissions using graphical tools**
  - [Describe database and object permissions](<2 - Implement a secure environment for a database service/1 - Configure database authentication and authorization/4 - Describe database and object permissions.md>)
- [ ] **Apply the principle of least privilege for all securables**
  - [Describe database and object permissions](<2 - Implement a secure environment for a database service/1 - Configure database authentication and authorization/4 - Describe database and object permissions.md>)
- [ ] **Troubleshoot authentication and authorization issues**
  - [Identify authentication and authorization failures](<2 - Implement a secure environment for a database service/1 - Configure database authentication and authorization/5 - Identify authentication and authorization failures.md>)
- [ ] **Manage authentication and authorization by using T-SQL**
  - [Describe Security Principals](<2 - Implement a secure environment for a database service/1 - Configure database authentication and authorization/3 - Describe Security Principals.md>) · [Describe database and object permissions](<2 - Implement a secure environment for a database service/1 - Configure database authentication and authorization/4 - Describe database and object permissions.md>)

### Implement security for data at rest and data in transit

- [ ] **Implement transparent data encryption (TDE)**
  - [Explore Transparent Data Encryption](<2 - Implement a secure environment for a database service/2 - Protect data in-transit and at rest/2 - Explore Transparent Data Encryption.md>) · [Understand Azure Key Vault](<2 - Implement a secure environment for a database service/2 - Protect data in-transit and at rest/7 - Understand Azure Key Vault.md>)
- [ ] **Implement object-level encryption**
  - [Explain object encryption and secure enclaves](<2 - Implement a secure environment for a database service/2 - Protect data in-transit and at rest/4 - Explain object encryption and secure enclaves.md>) · [Understand Azure Key Vault](<2 - Implement a secure environment for a database service/2 - Protect data in-transit and at rest/7 - Understand Azure Key Vault.md>)
- [ ] **Configure server- and database-level firewall rules**
  - [Configure server and database firewall rules](<2 - Implement a secure environment for a database service/2 - Protect data in-transit and at rest/3 - Configure server and database firewall rules.md>)
- [ ] **Implement Always Encrypted**
  - [Explain object encryption and secure enclaves](<2 - Implement a secure environment for a database service/2 - Protect data in-transit and at rest/4 - Explain object encryption and secure enclaves.md>)
- [ ] **Implement Always Encrypted with secure enclaves** — **changed 27 Oct 2026** (was "VBS enclaves")
  - [Always Encrypted with secure enclaves 🆕](<2 - Implement a secure environment for a database service/2 - Protect data in-transit and at rest/8 - [NEW] Always Encrypted with secure enclaves.md>)
- [ ] **Configure private links and service endpoints**
  - [Configure server and database firewall rules](<2 - Implement a secure environment for a database service/2 - Protect data in-transit and at rest/3 - Configure server and database firewall rules.md>) · [Enable encrypted connections](<2 - Implement a secure environment for a database service/2 - Protect data in-transit and at rest/5 - Enable encrypted connections.md>)

### Implement compliance controls for sensitive data

- [ ] **Apply a data classification strategy**
  - [Explore data classification](<2 - Implement a secure environment for a database service/3 - Implement compliance controls for sensitive data/2 - Explore data classification.md>) · [Implement Microsoft Purview](<2 - Implement a secure environment for a database service/3 - Implement compliance controls for sensitive data/9 - Implement Microsoft Purview.md>)
- [ ] **Configure server and database audits**
  - [Explore server and database audit](<2 - Implement a secure environment for a database service/3 - Implement compliance controls for sensitive data/3 - Explore server and database audit.md>)
- [ ] **Implement change data tracking**
  - [Change tracking and change data capture 🆕](<2 - Implement a secure environment for a database service/3 - Implement compliance controls for sensitive data/8 - [NEW] Change tracking and change data capture.md>)
- [ ] **Implement dynamic data masking**
  - [Implement Dynamic Data Masking](<2 - Implement a secure environment for a database service/3 - Implement compliance controls for sensitive data/4 - Implement Dynamic Data Masking.md>)
- [ ] **Implement ledger in Azure SQL**
  - [Explore Ledger](<2 - Implement a secure environment for a database service/3 - Implement compliance controls for sensitive data/7 - Explore Ledger.md>)
- [ ] **Implement row-level security**
  - [Implement Row Level security](<2 - Implement a secure environment for a database service/3 - Implement compliance controls for sensitive data/5 - Implement Row Level security.md>)

---

## 3. Monitor, configure, and optimize database resources (20–25%)


### Monitor resource activity and performance

- [ ] **Prepare an operational performance baseline**
  - [Establish baseline metrics](<3 - Monitor and optimize operational resources in Azure SQL/0 - Describe performance monitoring/4 - Establish baseline metrics.md>)
- [ ] **Determine sources for performance metrics**
  - [Describe performance monitoring tools](<3 - Monitor and optimize operational resources in Azure SQL/0 - Describe performance monitoring/2 - Describe performance monitoring tools.md>) · [Describe critical performance metrics](<3 - Monitor and optimize operational resources in Azure SQL/0 - Describe performance monitoring/3 - Describe critical performance metrics.md>)
- [ ] **Interpret performance metrics**
  - [Describe critical performance metrics](<3 - Monitor and optimize operational resources in Azure SQL/0 - Describe performance monitoring/3 - Describe critical performance metrics.md>)
- [ ] **Configure and monitor activity and performance**
  - [Describe performance monitoring tools](<3 - Monitor and optimize operational resources in Azure SQL/0 - Describe performance monitoring/2 - Describe performance monitoring tools.md>) · [Explore Query Performance Insight](<3 - Monitor and optimize operational resources in Azure SQL/0 - Describe performance monitoring/7 - Explore Query Performance Insight.md>)
- [ ] **Monitor by using database watcher**
  - [Describe database watcher (preview)](<3 - Monitor and optimize operational resources in Azure SQL/0 - Describe performance monitoring/6 - Describe database watcher (preview).md>)
- [ ] **Monitor by using Extended Events**
  - [Explore extended events](<3 - Monitor and optimize operational resources in Azure SQL/0 - Describe performance monitoring/5 - Explore extended events.md>)

### Monitor and optimize query performance

- [ ] **Configure Query Store**
  - [Explore Query Store](<4 - Optimize query performance in Azure SQL/1 - Explore query performance optimization/5 - Explore Query Store.md>) · [Diffrences](<Diffrences.md>)
- [ ] **Monitor by using Query Store**
  - [Explore Query Store](<4 - Optimize query performance in Azure SQL/1 - Explore query performance optimization/5 - Explore Query Store.md>) · [Understand query hints](<4 - Optimize query performance in Azure SQL/3 - Evaluate performance improvements/4 - Understand query hints.md>)
- [ ] **Identify and resolve session blocking**
  - [Describe blocking and locking](<4 - Optimize query performance in Azure SQL/1 - Explore query performance optimization/7 - Describe blocking and locking.md>)
- [ ] **Identify performance issues using dynamic management views (DMVs)**
  - [Describe dynamic management views and functions](<4 - Optimize query performance in Azure SQL/1 - Explore query performance optimization/4 - Describe dynamic management views and functions.md>) · [Azure SQL DMVs list](<Azure SQL DMVs list.md>)
- [ ] **Identify and implement index changes for queries**
  - [Design indexes](<4 - Optimize query performance in Azure SQL/2 - Explore performance-based database design/4 - Design indexes.md>) · [Tune and maintain indexes](<4 - Optimize query performance in Azure SQL/3 - Evaluate performance improvements/3 - Tune and maintain indexes.md>)
- [ ] **Recommend query construct modifications based on resource usage**
  - [Identify problematic query plans](<4 - Optimize query performance in Azure SQL/1 - Explore query performance optimization/6 - Identify problematic query plans.md>) · [Choose appropriate data types](<4 - Optimize query performance in Azure SQL/2 - Explore performance-based database design/3 - Choose appropriate data types.md>) · [Understand query hints](<4 - Optimize query performance in Azure SQL/3 - Evaluate performance improvements/4 - Understand query hints.md>)
- [ ] **Review execution plans**
  - [Understand query plans](<4 - Optimize query performance in Azure SQL/1 - Explore query performance optimization/2 - Understand query plans.md>) · [Explain estimated and actual query plans](<4 - Optimize query performance in Azure SQL/1 - Explore query performance optimization/3 - Explain estimated and actual query plans.md>) · [Identify problematic query plans](<4 - Optimize query performance in Azure SQL/1 - Explore query performance optimization/6 - Identify problematic query plans.md>) · [Diffrences](<Diffrences.md>)
- [ ] **Monitor by using Intelligent Insights**
  - [Intelligent Insights 🆕](<3 - Monitor and optimize operational resources in Azure SQL/0 - Describe performance monitoring/8 - [NEW] Intelligent Insights.md>)

### Configure database solutions for optimal performance

- [ ] **Implement index maintenance tasks**
  - [Explore database maintenance checks](<3 - Monitor and optimize operational resources in Azure SQL/2 - Configure databases for optimal performance/2 - Explore database maintenance checks.md>) · [Tune and maintain indexes](<4 - Optimize query performance in Azure SQL/3 - Evaluate performance improvements/3 - Tune and maintain indexes.md>)
- [ ] **Implement statistics maintenance tasks**
  - [Explore database maintenance checks](<3 - Monitor and optimize operational resources in Azure SQL/2 - Configure databases for optimal performance/2 - Explore database maintenance checks.md>)
- [ ] **Implement database integrity checks**
  - [Explore database maintenance checks](<3 - Monitor and optimize operational resources in Azure SQL/2 - Configure databases for optimal performance/2 - Explore database maintenance checks.md>) · [Create a SQL Server maintenance plan](<5 - Automate database tasks for Azure SQL/2 - Create and manage SQL Agent jobs/2 - Create a SQL Server maintenance plan.md>)
- [ ] **Configure database automatic tuning**
  - [Describe automatic tuning](<3 - Monitor and optimize operational resources in Azure SQL/2 - Configure databases for optimal performance/4 - Describe automatic tuning.md>)
- [ ] **Configure server settings for performance**
  - [Optimize database storage](<3 - Monitor and optimize operational resources in Azure SQL/1 - Configure SQL Server resources for optimal performance/4 - Optimize database storage.md>) · [Control SQL Server resources](<3 - Monitor and optimize operational resources in Azure SQL/1 - Configure SQL Server resources for optimal performance/5 - Control SQL Server resources.md>)
- [ ] **Configure Resource Governor for performance**
  - [Control SQL Server resources](<3 - Monitor and optimize operational resources in Azure SQL/1 - Configure SQL Server resources for optimal performance/5 - Control SQL Server resources.md>)
- [ ] **Implement database-scoped configuration**
  - [Describe database scoped configuration options](<3 - Monitor and optimize operational resources in Azure SQL/2 - Configure databases for optimal performance/3 - Describe database scoped configuration options.md>)
- [ ] **Configure compute and storage resources for scaling**
  - [Describe virtual machine resizing](<3 - Monitor and optimize operational resources in Azure SQL/1 - Configure SQL Server resources for optimal performance/3 - Describe virtual machine resizing.md>) · [Optimize database storage](<3 - Monitor and optimize operational resources in Azure SQL/1 - Configure SQL Server resources for optimal performance/4 - Optimize database storage.md>) · [Understand SQL database hyperscale](<1 - Plan and implement data platform resources/3 - Deploy PaaS solutions with Azure SQL/4 - Understand SQL database hyperscale.md>)
- [ ] **Identify use cases for intelligent query processing (IQP) features**
  - [Describe intelligent query processing](<3 - Monitor and optimize operational resources in Azure SQL/2 - Configure databases for optimal performance/5 - Describe intelligent query processing.md>)

---

## 4. Configure and manage automation of tasks (15–20%)


### Create and manage SQL Server Agent jobs

- [ ] **Manage schedules for regular maintenance jobs**
  - [Create a SQL Server maintenance plan](<5 - Automate database tasks for Azure SQL/2 - Create and manage SQL Agent jobs/2 - Create a SQL Server maintenance plan.md>)
- [ ] **Configure job alerts and notifications**
  - [Describe task status notifications](<5 - Automate database tasks for Azure SQL/2 - Create and manage SQL Agent jobs/3 - Describe task status notifications.md>)
- [ ] **Troubleshoot SQL Server Agent jobs**
  - [Troubleshoot SQL Server Agent jobs 🆕](<5 - Automate database tasks for Azure SQL/2 - Create and manage SQL Agent jobs/4 - [NEW] Troubleshoot SQL Server Agent jobs.md>)

### Automate deployment of database resources

- [ ] **Automate deployment by using ARM and Bicep templates**
  - [Automate deployment by using Azure Resource Manager templates and Bicep](<5 - Automate database tasks for Azure SQL/1 - Automate database deployment/3 - Automate deployment by using Azure Resource Manager templates and Bicep.md>)
- [ ] **Automate deployment by using Azure PowerShell**
  - [Automate deployment by using PowerShell](<5 - Automate database tasks for Azure SQL/1 - Automate database deployment/4 - Automate deployment by using PowerShell.md>)
- [ ] **Automate deployment by using Azure CLI**
  - [Automate deployment by using Azure CLI](<5 - Automate database tasks for Azure SQL/1 - Automate database deployment/5 - Automate deployment by using Azure CLI.md>)
- [ ] **Monitor and troubleshoot deployments**
  - [Monitor and troubleshoot deployments 🆕](<5 - Automate database tasks for Azure SQL/1 - Automate database deployment/6 - [NEW] Monitor and troubleshoot deployments.md>)

### Create and manage database tasks in Azure

- [ ] **Create and configure elastic jobs**
  - [Explore Elastic jobs](<5 - Automate database tasks for Azure SQL/3 - Manage Azure PaaS tasks using automation/2 - Explore Elastic jobs.md>)
- [ ] **Create and configure database tasks by using automation**
  - [Understand Azure Automation](<5 - Automate database tasks for Azure SQL/3 - Manage Azure PaaS tasks using automation/3 - Understand Azure Automation.md>) · [Build an automation runbook](<5 - Automate database tasks for Azure SQL/3 - Manage Azure PaaS tasks using automation/4 - Build an automation runbook.md>) · [Automate database workflows by using Logic Apps](<5 - Automate database tasks for Azure SQL/3 - Manage Azure PaaS tasks using automation/5 - Automate database workflows by using Logic Apps.md>)
- [ ] **Configure alerts and notifications on database tasks**
  - [Monitor automated tasks](<5 - Automate database tasks for Azure SQL/3 - Manage Azure PaaS tasks using automation/6 - Monitor automated tasks.md>) · [Describe task status notifications](<5 - Automate database tasks for Azure SQL/2 - Create and manage SQL Agent jobs/3 - Describe task status notifications.md>)
- [ ] **Troubleshoot automated database tasks**
  - [Monitor automated tasks](<5 - Automate database tasks for Azure SQL/3 - Manage Azure PaaS tasks using automation/6 - Monitor automated tasks.md>)

---

## 5. Plan and configure a high availability and disaster recovery (HA/DR) environment (20–25%)


### Plan an HA/DR strategy for database solutions

- [ ] **Recommend HA/DR strategy based on RPO/RTO requirements**
  - [Describe recovery time objective and recovery point objective](<6 - Plan and implement a high availability and disaster recovery environment/1 - Describe high availability and disaster recovery strategies/2 - Describe recovery time objective and recovery point objective.md>) · [Explore high availability and disaster recovery options](<6 - Plan and implement a high availability and disaster recovery environment/1 - Describe high availability and disaster recovery strategies/3 - Explore high availability and disaster recovery options.md>)
- [ ] **Evaluate HA/DR for hybrid deployments**
  - [Describe hybrid solutions](<6 - Plan and implement a high availability and disaster recovery environment/1 - Describe high availability and disaster recovery strategies/7 - Describe hybrid solutions.md>)
- [ ] **Evaluate Azure-specific HA/DR solutions**
  - [Describe Azure high availability and disaster recovery features for Azure Virtual Machines](<6 - Plan and implement a high availability and disaster recovery environment/1 - Describe high availability and disaster recovery strategies/4 - Describe Azure high availability and disaster recovery features for Azure Virtual Machines.md>) · [Describe high availability and disaster recovery for PaaS deployments](<6 - Plan and implement a high availability and disaster recovery environment/1 - Describe high availability and disaster recovery strategies/5 - Describe high availability and disaster recovery for PaaS deployments.md>) · [Explore high availability and disaster recovery solution for IaaS](<6 - Plan and implement a high availability and disaster recovery environment/1 - Describe high availability and disaster recovery strategies/6 - Explore high availability and disaster recovery solution for IaaS.md>) · [Explain high availability and disaster recovery options](<1 - Plan and implement data platform resources/2 - Deploy IaaS solutions with Azure SQL/5 - Explain high availability and disaster recovery options.md>)
- [ ] **Plan a testing procedure for an HA/DR solution**
  - [Test and troubleshoot HA-DR solutions 🆕](<6 - Plan and implement a high availability and disaster recovery environment/2 - Explore IaaS and PaaS solutions for high availability and disaster recovery/7 - [NEW] Test and troubleshoot HA-DR solutions.md>)

### Plan and perform backup and restore of a database

- [ ] **Recommend a database backup and restore strategy**
  - [Back up and restore SQL Server running on Azure virtual machines](<6 - Plan and implement a high availability and disaster recovery environment/3 - Back up and restore databases/2 - Back up and restore SQL Server running on Azure virtual machines.md>) · [Back up and restore a database for SQL Database and SQL Managed Instance](<6 - Plan and implement a high availability and disaster recovery environment/3 - Back up and restore databases/4 - Back up and restore a database for SQL Database and SQL Managed Instance.md>)
- [ ] **Perform a database backup by using native tools**
  - [Back up and restore SQL Server running on Azure virtual machines](<6 - Plan and implement a high availability and disaster recovery environment/3 - Back up and restore databases/2 - Back up and restore SQL Server running on Azure virtual machines.md>) · [Back up a SQL Server virtual machine](<6 - Plan and implement a high availability and disaster recovery environment/3 - Back up and restore databases/3 - Back up a SQL Server virtual machine.md>)
- [ ] **Perform a database restore by using native tools**
  - [Back up and restore SQL Server running on Azure virtual machines](<6 - Plan and implement a high availability and disaster recovery environment/3 - Back up and restore databases/2 - Back up and restore SQL Server running on Azure virtual machines.md>) · [Back up a SQL Server virtual machine](<6 - Plan and implement a high availability and disaster recovery environment/3 - Back up and restore databases/3 - Back up a SQL Server virtual machine.md>)
- [ ] **Perform a database restore to a point in time**
  - [Back up and restore a database for SQL Database and SQL Managed Instance](<6 - Plan and implement a high availability and disaster recovery environment/3 - Back up and restore databases/4 - Back up and restore a database for SQL Database and SQL Managed Instance.md>) · [Back up and restore SQL Server running on Azure virtual machines](<6 - Plan and implement a high availability and disaster recovery environment/3 - Back up and restore databases/2 - Back up and restore SQL Server running on Azure virtual machines.md>)
- [ ] **Configure long-term backup retention**
  - [Back up and restore a database for SQL Database and SQL Managed Instance](<6 - Plan and implement a high availability and disaster recovery environment/3 - Back up and restore databases/4 - Back up and restore a database for SQL Database and SQL Managed Instance.md>)
- [ ] **Backup and restore a database by using T-SQL**
  - [Back up and restore SQL Server running on Azure virtual machines](<6 - Plan and implement a high availability and disaster recovery environment/3 - Back up and restore databases/2 - Back up and restore SQL Server running on Azure virtual machines.md>) · [Back up a SQL Server virtual machine](<6 - Plan and implement a high availability and disaster recovery environment/3 - Back up and restore databases/3 - Back up a SQL Server virtual machine.md>)
- [ ] **Backup to and restore from cloud storage**
  - [Back up and restore SQL Server running on Azure virtual machines](<6 - Plan and implement a high availability and disaster recovery environment/3 - Back up and restore databases/2 - Back up and restore SQL Server running on Azure virtual machines.md>) · [Migrate databases using native backup and restore](<1 - Plan and implement data platform resources/Migrate SQL Server workloads to Azure Virtual Machine/6 -Migrate databases using native backup and restore.md>)

### Configure HA/DR for database solutions

- [ ] **Configure active geo-replication**
  - [Describe active geo-replication for Azure SQL Database](<6 - Plan and implement a high availability and disaster recovery environment/2 - Explore IaaS and PaaS solutions for high availability and disaster recovery/4 - Describe active geo-replication for Azure SQL Database.md>)
- [ ] **Configure Always On availability groups on SQL Managed Instance and Azure VMs**
  - [Configure Always-on availability groups](<6 - Plan and implement a high availability and disaster recovery environment/2 - Explore IaaS and PaaS solutions for high availability and disaster recovery/3 - Configure Always-on availability groups.md>) · [Explore high availability and disaster recovery solution for IaaS](<6 - Plan and implement a high availability and disaster recovery environment/1 - Describe high availability and disaster recovery strategies/6 - Explore high availability and disaster recovery solution for IaaS.md>) · [Migrate using Managed Instance link](<1 - Plan and implement data platform resources/5 - Migrate SQL Server workloads to Azure SQL Managed Instance/4 - Migrate using Managed Instance link.md>)
- [ ] **Configure failover groups**
  - [Explore auto-failover groups for Azure SQL Database and Azure SQL Managed Instance](<6 - Plan and implement a high availability and disaster recovery environment/2 - Explore IaaS and PaaS solutions for high availability and disaster recovery/5 - Explore auto-failover groups for Azure SQL Database and Azure SQL Managed Instance.md>)
- [ ] **Configure Always On Failover Cluster Instances on Azure VMs**
  - [Describe failover clusters in Windows Server](<6 - Plan and implement a high availability and disaster recovery environment/2 - Explore IaaS and PaaS solutions for high availability and disaster recovery/2 - Describe failover clusters in Windows Server.md>) · [Explore high availability and disaster recovery solution for IaaS](<6 - Plan and implement a high availability and disaster recovery environment/1 - Describe high availability and disaster recovery strategies/6 - Explore high availability and disaster recovery solution for IaaS.md>)
- [ ] **Configure log shipping**
  - [Explore high availability and disaster recovery options](<6 - Plan and implement a high availability and disaster recovery environment/1 - Describe high availability and disaster recovery strategies/3 - Explore high availability and disaster recovery options.md>) · [Explore high availability and disaster recovery solution for IaaS](<6 - Plan and implement a high availability and disaster recovery environment/1 - Describe high availability and disaster recovery strategies/6 - Explore high availability and disaster recovery solution for IaaS.md>)
- [ ] **Monitor an HA/DR solution**
  - [Monitor availability](<6 - Plan and implement a high availability and disaster recovery environment/2 - Explore IaaS and PaaS solutions for high availability and disaster recovery/6 - Monitor availability.md>) · [Test and troubleshoot HA-DR solutions 🆕](<6 - Plan and implement a high availability and disaster recovery environment/2 - Explore IaaS and PaaS solutions for high availability and disaster recovery/7 - [NEW] Test and troubleshoot HA-DR solutions.md>)
- [ ] **Troubleshoot an HA/DR solution**
  - [Test and troubleshoot HA-DR solutions 🆕](<6 - Plan and implement a high availability and disaster recovery environment/2 - Explore IaaS and PaaS solutions for high availability and disaster recovery/7 - [NEW] Test and troubleshoot HA-DR solutions.md>)
