## 🩹 **Who Patches What? (Shared responsibility)**

| Deployment                                   | Who patches OS + SQL Server?          | Your controls                                                                 |
| -------------------------------------------- | ------------------------------------- | ----------------------------------------------------------------------------- |
| **SQL Server on Azure VM** (IaaS)            | **You**                               | Azure Update Manager (recommended), Automated Patching (retiring), manual CUs |
| **SQL Server enabled by Azure Arc** (hybrid) | **You**                               | Arc **automatic updates** setting, **Azure Update Manager**                    |
| **Azure SQL Managed Instance** (PaaS)        | **Microsoft**                         | **Maintenance window**, **update policy**, advance notifications              |
| **Azure SQL Database** (PaaS)                | **Microsoft**                         | **Maintenance window**, advance notifications                                 |

- DP-300 skill: **"Apply patches and updates for hybrid and IaaS deployments"**. Know the **tools**, which **update types** each installs, and how to patch **HA clusters without downtime**.

---

## ⏳ **Automated Patching (SQL IaaS Agent extension) - RETIRING**

- Feature of the **SQL Server IaaS Agent extension** (SQL virtual machines resource → **Updates**).
- Sets a **maintenance window**: day (or Everyday), **start hour**, and **duration of 30–180 minutes**.
- Installs **only Windows and SQL Server updates marked Important or Critical**.
  - ❌ It does **NOT** install **cumulative updates (CUs)** or service packs that aren't marked Important/Critical.
- Windows only. Supported from SQL Server 2012 on Windows Server 2012.
- **Not AG-aware**: give each replica a **different day/time**, or both nodes may reboot together.
- ⚠️ **Status: scheduled to retire on 17 September 2027.** Microsoft says to **avoid it for new environments** and **migrate to Azure Update Manager**. On exam day (Nov 2026) it **still works, but is deprecated**.

```powershell
# Legacy: configure Automated Patching (Thursday 11:00, 120 min)
Update-AzSqlVM -ResourceGroupName 'rg-sql' -Name 'sqlvm01' `
  -AutoPatchingSettingDayOfWeek Thursday `
  -AutoPatchingSettingMaintenanceWindowStartingHour 11 `
  -AutoPatchingSettingMaintenanceWindowDuration 120 `
  -AutoPatchingSettingEnable
```

---

## 🛠️ **Azure Update Manager (the replacement)**

- One service to patch **Azure VMs, Arc-enabled servers (on-prem/other clouds), VMware/SCVMM machines** for both **Windows and Linux**.
- **Native**: it has **no dependency on Log Analytics or Azure Automation**. It replaced the old *Automation Update Management*.
- For SQL VMs it **installs SQL Server cumulative updates** (Automated Patching can't), plus Critical/Important OS and SQL updates.
- Key capabilities:
  - **Periodic assessment**: checks for missing updates **every 24 hours** and reports compliance.
  - **One-time (on-demand) updates**: filter by classification, KB ID or publish date.
  - **Scheduled updates**: a **maintenance configuration** (schedule + classifications) assigned to many VMs; supports **dynamic scoping** and **Azure Policy**.
  - **Hotpatching** for supported Windows Server Azure Edition VMs (no reboot for many security updates).
- Enable it from the SQL virtual machines resource:
  1. **Updates** → **Disable** Automated Patching.
  2. Select **Try Azure Update Manager**.
  3. **Settings** → **Enable Microsoft updates**. This is required to receive **SQL Server** updates, not just Windows ones.
  4. For schedules, set **Patch orchestration = Customer Managed Schedules** and include the **Updates** classification to get **CUs**.
- ⚠️ Use **either** Automated Patching **or** Update Manager, **never both**. Overlapping tools cause scheduling conflicts and patches outside the window. The same applies to Automatic VM guest patching.
- ⚠️ Update Manager is **also not AG-aware**. Put AG replicas in **different maintenance configurations** or windows.

```powershell
# Official migration path: converts the Automated Patching schedule to an Update Manager maintenance configuration
Install-Module -Name MigrateSQLVMPatchingSchedule-Module -Force -AllowClobber
Import-Module MigrateSQLVMPatchingSchedule-Module
Convert-SQLVMPatchingSchedule -ResourceGroupName 'rg-sql' -VmName 'sqlvm01'
# Disables Automated Patching, enables Microsoft Update, creates + assigns a maintenance configuration
```

---

## 🌐 **Hybrid: SQL Server Enabled by Azure Arc**

- **Option 1 - Arc automatic updates** (Server - Azure Arc → **SQL Server Configuration** → **Update**):
  - Sets a weekly maintenance schedule + start hour, and configures **Windows Update / Microsoft Update** on the host.
  - **Important/Critical updates only**. No CUs; install those manually or via Update Manager.
  - **Windows hosts only**, and applies at OS level to **all instances on that host**.
  - Only for license type **Software Assurance or pay-as-you-go**. It's **disabled for "License only"**.
- **Option 2 - Azure Update Manager** for Arc-enabled servers: the same schedules, assessments and policy as for Azure VMs. Enable Microsoft updates to include SQL Server.
- **Extended Security Updates (ESU)** for end-of-support SQL versions can be bought and delivered through Arc.

---

## 🔁 **Patching HA Configurations (Rolling Updates)**

**Always On availability group:** downtime = **one manual failover** (two if you fail back).

1. **Remove automatic failover** on all synchronous-commit replicas.
2. Patch **asynchronous** secondaries → **remote** sync secondaries → **local** sync secondaries.
3. Check the target is **SYNCHRONIZED**, then **manually fail over** to a patched **synchronous** secondary.
4. Patch the **former primary**.
5. Restore automatic failover; optionally fail back.

```sql
-- Before patching: make failover manual and confirm sync state
ALTER AVAILABILITY GROUP [AG1] MODIFY REPLICA ON 'SQLVM02'
    WITH (FAILOVER_MODE = MANUAL);

SELECT ar.replica_server_name, drs.synchronization_state_desc
FROM sys.dm_hadr_database_replica_states drs
JOIN sys.availability_replicas ar ON drs.replica_id = ar.replica_id;

-- On the patched synchronous secondary that becomes the new primary
ALTER AVAILABILITY GROUP [AG1] FAILOVER;
```

- Never patch the **primary first**: an upgraded primary **can't ship log** to older secondaries.
- Point backup preference at the **primary** while secondaries are being patched.
- **Cluster-Aware Updating (CAU) is NOT supported** for AGs. Mixed versions should exist only **briefly**.
- **Failover cluster instance (FCI)**: patch the **passive node(s)** first, **fail over**, then patch the old active node.
- **CDC/replication** databases: after failing over to an upgraded instance, run `sp_vupgrade_replication` if the update needs it. Not needed from SQL Server 2019 CU1 onward.

---

## ☁️ **PaaS: Microsoft Patches, You Choose When / Which Version**

**Maintenance window (Azure SQL Database & Managed Instance)**

| Option                  | When                                                     |
| ----------------------- | -------------------------------------------------------- |
| **System default**      | Most impactful updates blocked **8 AM–5 PM local time** (urgent ones can still happen) |
| **Weekday**             | **10 PM–6 AM**, Monday–Thursday starts                   |
| **Weekend**             | **10 PM–6 AM**, Friday–Sunday starts                     |

- Free to configure. Changing it ends with a short reconfiguration (**~8 s**). Not available for SQL DB **Basic/S0/S1**, DC hardware, or **MI instance pools**. Not available for **Dev/Test** subscription offers.
- **Advance notifications** (Azure Monitor alerts) **24 h before**, at start and at end. For SQL DB they require a **non-default** window.
- It only covers **planned** maintenance. Hardware failures and SLO changes can still interrupt.
- **Failover groups / geo-replicas not on paired regions** → choose **different windows** for primary and secondary.
- MI: changing the window **moves the instance to a different VM group and changes its IP address**, so check NSG/firewall rules.
- Use the **Redirect** connection policy to minimize disruption.

**Managed Instance update policy** (Settings → **Maintenance and updates**)

| Policy                 | Engine format                    | Key point                                                                              |
| ---------------------- | -------------------------------- | -------------------------------------------------------------------------------------- |
| **SQL Server 2022**    | Aligned with SQL 2022            | Restore DB **to SQL Server 2022**, MI link bidirectional failover with 2022             |
| **SQL Server 2025**    | Aligned with SQL 2025            | **Default for new instances via portal** (REST 2026-08-01-preview+); restore/link with 2025 |
| **Always-up-to-date**  | Evolves continuously ("Continuous") | All newest features; **can't go back**; no restore to boxed SQL Server              |

- Upgrading the policy **permanently upgrades the database format**. The only way is up: 2022 → 2025 → Always-up-to-date.
- `SELECT SERVERPROPERTY('ProductUpdateType');` → `CU` (2022/2025 policy) or `Continuous` (Always-up-to-date).
- PowerShell/CLI still default to **SQL Server 2022**, so set `DatabaseFormat` / `--database-format` explicitly in templates.

---

## ✅ **Summary**

- **IaaS & Arc = you patch.** Use **Azure Update Manager**: schedules, 24 h assessments, **CUs**, Windows + Linux, Azure + Arc.
- **Automated Patching** (SQL IaaS Agent ext.) = Important/Critical only, **no CUs**, **retires 17 Sep 2027**.
- Arc automatic updates = Important/Critical only, Windows only, requires **SA or PAYG** license type.
- HA patching = **rolling**: secondaries first, manual failover to a synchronous secondary, then the old primary. **No CAU for AGs.**
- **PaaS = Microsoft patches.** You control **when** (maintenance window + notifications) and, on MI, **which engine line** (update policy).

---

## 🎯 **Exam Tips / Traps**

- "Automatically install **SQL Server cumulative updates** on Azure VMs on a schedule" → **Azure Update Manager** (not Automated Patching).
- "Patch on-prem + multi-cloud SQL Servers from one place" → **Arc-enable** them + **Azure Update Manager**.
- "Least effort to keep patching after **Automated Patching retirement**" → `Convert-SQLVMPatchingSchedule` (MigrateSQLVMPatchingSchedule module).
- "Both AG nodes rebooted at the same time" → patch schedules overlapped. Neither Automated Patching nor Update Manager is AG-aware.
- "Minimize AG downtime during a CU install" → **rolling update** with **one manual failover**, after setting failover mode to **manual**.
- "Prevent planned maintenance during business hours on SQL DB/MI" → **maintenance window** (Weekday/Weekend), plus **advance notifications**.
- "Need to restore an MI database back to on-prem SQL Server 2022" → keep the **SQL Server 2022 update policy**. Once on **Always-up-to-date** you can't go back.
- Arc automatic updates greyed out → the license type is **License only**. Change it to **PAYG or SA**.

> Source: https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/automated-patching · https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/azure-update-manager-sql-vm · https://learn.microsoft.com/en-us/azure/update-manager/overview · https://learn.microsoft.com/en-us/azure/update-manager/guidance-patching-sql-server-azure-vm · https://learn.microsoft.com/en-us/sql/sql-server/azure-arc/update · https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/upgrading-always-on-availability-group-replica-instances · https://learn.microsoft.com/en-us/azure/azure-sql/database/maintenance-window · https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/maintenance-window · https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/update-policy
