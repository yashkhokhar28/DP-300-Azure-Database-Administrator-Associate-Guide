## ✅ **3 Main Azure HA/DR Options for Virtual Machines**

| Feature                 | Protects Against                                             | Example Usage                             |
| ----------------------- | ------------------------------------------------------------ | ----------------------------------------- |
| **Availability Sets**   | Hardware and maintenance failures within a single datacenter | Keep VMs on separate servers              |
| **Availability Zones**  | Full datacenter failure in a region                          | Place VMs in different datacenters        |
| **Azure Site Recovery** | Entire region failures                                       | Replicate VMs to a different Azure region |

---

## 🔹 **Availability Sets: Separate VMs in a Single Datacenter**

- Protects against **single points of failure** and Azure platform maintenance **in one datacenter.**
- Works by separating VMs across:

  - **Fault Domains (FD):** Protect from hardware, power, or network failures.
  - **Update Domains (UD):** Protect from reboots during Azure maintenance.

Example: If you have **two SQL Server VMs**, an availability set ensures they **won’t be on the same physical server**, preventing both from failing at once during hardware failure or updates.

➡️ **Limitations:**

- Does **not protect against SQL Server crashes** or OS issues inside the VM.
- Only protects **within one datacenter.**
- Cannot be combined with Availability Zones.

➡️ **Best practice:**
For **multi-tier apps**, use separate availability sets for each tier (Web servers, SQL Server, AD DS, etc.).

---

## 🔹 **Availability Zones: Protect Against Entire Datacenter Failure**

- Spreads your VMs across **physically separate datacenters** (**Zones 1, 2, 3**) within the same Azure region.
- Each zone has its own power, network, and cooling.
- Provides **much higher availability** than availability sets.

➡️ **Key points:**

- Zones are **physically separate**, but logically represented (e.g., your Zone 1 VM and another user's Zone 1 VM may be in different datacenters).
- Some **network latency** (usually < 1 ms), which still supports **synchronous replication** like Availability Groups.
- You **cannot mix Availability Zones and Availability Sets**.

➡️ **Used for:**
Protecting apps from **data center-wide outages** within the same Azure region.

---

## 🔹 **Azure Site Recovery: Disaster Recovery Across Regions**

- Creates a **disaster recovery (DR) plan** by **replicating VMs from one Azure region to another.**
- If an entire region fails, you can **failover to the replica region.**

➡️ **Key limitations:**

- Azure Site Recovery **doesn’t understand what’s running inside the VM** (like SQL Server transactions).
- RTO and RPO are **at the VM level**, not the SQL Server/database level.
- RTO (Recovery Time Objective) is generally **within 2 hours**, but RPO (Recovery Point Objective) depends on how often replication occurs.

➡️ **Used for:**

- Simpler DR for whole VMs when **SQL-specific DR tools (like Availability Groups) are too complex or not required**.
- Meeting business DR goals at the infrastructure level.

---

## ⚠️ **Important Clarifications:**

- Availability Sets/Zones and Site Recovery only protect **the infrastructure (VMs, networks, storage)**.
- You still need **SQL Server HA/DR tools (AGs, FCIs, Log Shipping)** to protect:

  - SQL Server failures
  - Database-level RTO/RPO

---

## ✔️ Summary Table

| Azure Feature       | Scope              | Protects Against                  | Example Use                                          |
| ------------------- | ------------------ | --------------------------------- | ---------------------------------------------------- |
| Availability Set    | Same datacenter    | Hardware and maintenance failures | SQL VMs on separate servers                          |
| Availability Zone   | Across datacenters | Datacenter failures in a region   | Primary and secondary SQL Servers in different zones |
| Azure Site Recovery | Across regions     | Regional disaster recovery        | Failover entire VMs to another region                |

---
