## 🔑 **Overview of the 5 Example Solutions**

| Example Type             | Architecture                              | Purpose                              |
| ------------------------ | ----------------------------------------- | ------------------------------------ |
| HA (1 Region)            | Always On Availability Groups (AG)        | High availability within a region    |
| HA (1 Region)            | Always On Failover Cluster Instance (FCI) | High availability within a region    |
| DR (Multi-Region/Hybrid) | Always On Availability Groups             | Disaster recovery across regions     |
| DR (Multi-Region/Hybrid) | Distributed Availability Groups           | Scalable DR across multiple clusters |
| DR (Multi-Region/Hybrid) | Log Shipping or Azure Site Recovery       | Simpler or infrastructure-level DR   |

---

## ▶️ **Single Region High Availability**

### 1. **Always On Availability Groups (AG)**

- **What it does:** Protects **databases** using **multiple synchronized replicas** on separate VMs.
- **Why use it:**

  - No shared storage needed.
  - Fast failover.
  - Meets low RTO/RPO with minimal data loss.
  - Supports read-only secondaries for reporting.

- **Limitation:** Doesn’t protect instance-level objects (logins, jobs, etc.).

---

### 2. **Always On Failover Cluster Instance (FCI)**

- **What it does:** Protects the **entire SQL Server instance**, not just databases.
- **Why use it:**

  - Well-known technology.
  - Protects all instance-level objects.
  - Good for apps that expect the whole SQL Server to fail over.
  - Modern shared storage solutions like **Azure Shared Disk** improve FCI in Azure.

- **Limitation:** Failover takes longer than AG. Less needed in VMs where hardware failure is rare.

---

## ▶️ **Multi-Region Disaster Recovery**

### 3. **Always On Availability Group Across Regions (or Hybrid)**

- **What it does:** AG spans **multiple Azure regions** or between Azure and on-premises.
- **Why use it:**

  - Proven method similar to traditional multi-datacenter designs.
  - One feature provides both **HA & DR**.
  - Can work in **Standard or Enterprise Edition**.

- **Considerations:**

  - Requires **good network connectivity**.
  - Requires Active Directory and DNS in all regions.
  - Witness placement is critical to avoid split-brain scenarios.

---

### 4. **Distributed Availability Group**

- **What it does:**

  - Introduced in SQL Server 2016 Enterprise.
  - It's like an **AG of AGs**, where **each cluster has its own quorum and witness**.
  - Provides better separation and scalability than a single AG stretched across regions.

- **Why use it:**

  - Avoids a single point of failure in the WSFC.
  - Each region (or cluster) manages itself.
  - Can replicate across cloud regions or from on-prem to Azure.
  - Simplifies **failback** between locations.

---

### 5. **Log Shipping**

- **What it does:** Copies transaction log backups to a secondary server and restores them, creating a **warm standby**.
- **Why use it:**

  - **Simple, reliable, and time-tested.**
  - Works even on poor network connections.
  - Meets many DR goals.
  - Helps protect FCIs (as they don't replicate data themselves).

- **Limitations:**

  - **Manual failover.**
  - Some **data loss is expected** unless failover is pre-planned.

---

## ▶️ **Disaster Recovery with Azure Site Recovery (ASR)**

- **What it does:** Replicates entire VMs to another Azure region (or to Azure from on-prem).
- **Why use it:**

  - Works with **many workloads**, not just SQL Server.
  - Easy to configure through the Azure Portal.
  - Meets some RTO/RPO needs, though database professionals usually prefer AG or log shipping for better **data-level control**.

- **Limitations:**

  - Doesn’t understand **SQL Server transactions**, so it might not meet strict RPO needs.

---

## ✔️ **Summary: When to Use What**

| Scenario                                   | Recommended Solution              |
| ------------------------------------------ | --------------------------------- |
| High availability in 1 Azure region        | Always On AG or FCI               |
| Disaster recovery between Azure regions    | Multi-region AG or distributed AG |
| Disaster recovery from on-prem to Azure    | Distributed AG or log shipping    |
| Simpler VM-level disaster recovery         | Azure Site Recovery               |
| Need to protect instance-level objects too | FCI                               |
| Minimal cost DR with some manual work      | Log shipping                      |

---
