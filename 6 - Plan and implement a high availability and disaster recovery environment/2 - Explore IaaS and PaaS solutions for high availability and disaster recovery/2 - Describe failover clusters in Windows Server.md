## 🔑 **Key Concepts in Failover Clustering on Azure**

| Term                                          | Meaning                                                                                   |
| --------------------------------------------- | ----------------------------------------------------------------------------------------- |
| **WSFC**                                      | A Windows cluster that manages failover between SQL Server instances or databases.        |
| **Witness Resource**                          | A resource that helps decide which nodes stay online during a failure (maintains quorum). |
| **Cloud Witness**                             | Azure-based witness, recommended for Azure and hybrid clusters.                           |
| **Quorum**                                    | A voting system that decides whether the cluster can stay operational.                    |
| **Active Directory (AD DS)**                  | Needed for traditional WSFCs. Optional for some newer setups using Workgroup Clusters.    |
| **DNS**                                       | Required for resolving cluster and service names in the network.                          |
| **DTC (Distributed Transaction Coordinator)** | A Windows component that some apps require. Needs special setup in clusters.              |

---

## ⚙️ **Special Azure Considerations for WSFC**

Unlike on-premises clusters, Azure WSFCs require adjustments:

- **Witness resource:** Use **Cloud Witness** instead of a disk or file share for reliability across regions.
- **IP addresses:** Must be configured **in Azure**, not in the VM. The VMs see their IPs as DHCP-assigned.
- **Single vNIC:** Usually only one network interface per VM in Azure clusters.
- **No static IP reservation:** Unlike on-premises, Azure DHCP manages IPs dynamically — check for address conflicts manually.
- **Cluster Name Object (CNO):** Created in AD DS for traditional WSFCs. Workgroup Clusters don't need AD DS but still need DNS entries.

---

## 🔧 **Setting Up WSFC in Azure**

### 1. **Enable Failover Clustering Feature**

Install on each node using:

```powershell
Install-WindowsFeature Failover-Clustering -IncludeManagementTools
```

### 2. **Validate the Cluster**

Run validation before creating the cluster to ensure the environment is properly configured:

```powershell
Test-Cluster
```

Warnings (e.g., about missing shared storage in AGs) can be expected in Azure.

### 3. **Create the Cluster**

Use **PowerShell**, not the GUI, to correctly handle Azure’s networking quirks. Example for no shared storage:

```powershell
New-Cluster -Name MyWSFC -Node Node1,Node2 -StaticAddress 10.0.0.5 -NoStorage
```

You can create **Workgroup Clusters** with:

```powershell
New-Cluster -Name MyWSFC -Node Node1,Node2 -StaticAddress 10.0.0.5 -NoStorage -AdministrativeAccessPoint DNS
```

From **Windows Server 2019**, by default, Azure clusters use a **Distributed Network Name (DNN)** instead of a Virtual Network Name (VNN), simplifying networking.

---

## 🛡️ **Failover Cluster Instance (FCI) Basics**

- FCI protects at the **instance level** (the whole SQL Server installation, not just databases).
- Clients connect to a **virtual name**, so they don’t know or care which node SQL Server is running on.
- Supports **multi-subnet failover** (Azure regions or availability zones):

  - Each subnet has its own IP.
  - During failover, DNS updates the virtual name to point to the right IP.

From Windows Server 2019 onward, **DNN replaces VNN for FCI**, removing the need for Azure Load Balancers.

---

## 🔍 **Testing the Cluster**

Before setting up SQL features, verify:

- The cluster works.
- If using shared disks (**for FCIs**), confirm that each node can fail over and access the disks.

You test this using **Failover Cluster Manager → Storage → Disks → Move Available Storage**.

---

## ✅ **Summary of Key Points**

- You must adjust how you deploy WSFC on Azure compared to on-premises.
- **Use PowerShell**, not the GUI, to create Azure clusters.
- Consider **Cloud Witness** and **DNN** for simplicity and Azure optimization.
- Validate your setup before proceeding with Always On Availability Groups or FCIs.

---

### ▶️ Next Topic

> **Configure Always On Availability Groups on top of the WSFC.**
