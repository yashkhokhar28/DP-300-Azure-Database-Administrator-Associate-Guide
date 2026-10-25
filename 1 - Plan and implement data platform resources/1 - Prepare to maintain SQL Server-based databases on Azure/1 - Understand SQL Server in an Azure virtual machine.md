### **SQL Server in Azure Virtual Machines (IaaS):**

- Azure VMs running SQL Server work like on-premises SQL Server installations.
- Suitable when applications require:

  - Older SQL Server versions for compatibility.
  - Additional services like SSAS, SSIS, SSRS on the same machine.

---

### **Available Versions:**

- Microsoft provides Azure Marketplace images for supported versions.
- Older versions need manual installation if under extended support.
- Marketplace images today: SQL Server **2016 → 2025** (2025 on Windows Server 2025). SQL Server **2012/2014** images are no longer in the Marketplace. 🔄

---

### **Backup Options:**

- **Backup to URL:** Stores backups in Azure Blob Storage.
- **Azure Backup for SQL VMs:** Manages backups across infrastructure automatically.

---

### **Deployment Options:**

- Azure Resource Manager (ARM) templates use a **declarative** approach (preferred for large deployments).
- Other deployment methods are **imperative**.

---

### **Azure Storage Types for SQL Server:**

- **Premium SSD / Premium SSD v2 / Ultra Disk:** Recommended for data and transaction logs (low latency). Premium SSD v2 gives the best price-performance for most SQL workloads (e.g., Ebdsv5 VMs). 🔄
- **Standard storage:** Suitable for database backups.

---

### **High Availability:**

- Azure VMs have high fault tolerance, with 99.9% uptime guaranteed (when using Premium SSD or Ultra Disk).
- High availability options include:

  - Availability sets
  - Availability zones
  - Load balancing

---

### **SQL Server with Azure Arc:**

- Azure Arc extends Azure management to SQL Servers outside Azure.
- Benefits:

  - Centralized management and monitoring.
  - Consistent policy enforcement and compliance.
  - Enhanced security and performance insights.
  - Supports automated backups (preview), disaster recovery, and automatic updates. 🔄
  - Pay-as-you-go SQL Server licensing billed through Azure, and **Extended Security Updates (ESU)** for end-of-support versions. 🔄
  - Enables use of Azure AI/ML capabilities with your SQL Server data.

---
