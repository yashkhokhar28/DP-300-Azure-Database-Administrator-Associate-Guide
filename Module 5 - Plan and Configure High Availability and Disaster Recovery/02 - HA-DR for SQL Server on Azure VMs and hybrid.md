# HA/DR for SQL Server on Azure VMs and hybrid

> **Exam skills:** Evaluate Azure-specific HA/DR solutions, Evaluate HA/DR for hybrid deployments, Configure Always On Failover Cluster Instances on Azure VMs, Configure Always On availability groups on SQL Managed Instance and Azure VMs, Configure log shipping
> **Reading time:** ~17 min

## The big idea
On a VM, Azure keeps the hardware running, but SQL Server is your job. You use two layers. **Azure features** (availability sets, zones, Site Recovery) protect the VMs. **SQL Server features** (FCI, AG, log shipping) protect SQL Server and the data. You usually need both.

Think of a school bus company. Azure gives you spare buses parked in different garages. But you still need a trained driver and a plan to move the children to the spare bus.

## Azure features that protect the VM
| Feature | Protects against | Scope |
|---|---|---|
| **Availability set** | Host hardware failure and Azure maintenance reboots | One datacenter |
| **Availability zone** | A whole datacenter failing | Across datacenters in one region |
| **Azure Site Recovery (ASR)** | A whole region failing | Another region |

### Availability sets
- Spread VMs across **fault domains** (separate power, network and racks) and **update domains** (groups that are rebooted at different times during maintenance).
- Two SQL Server VMs in one set will not sit on the same physical server.
- Use a separate availability set for each tier (web, SQL, domain controllers).
- Does **not** protect against SQL Server or Windows problems inside the VM.

### Availability zones
- Zones 1, 2 and 3 are **physically separate datacenters** in one region, each with its own power, cooling and network.
- Higher availability than sets.
- Round-trip latency between zones is low (Microsoft targets below about 2 ms), so **synchronous** AG replicas still work across zones.
- You **can't combine** availability zones and availability sets for the same VM.

**Example:** A hospital puts SQL VM 1 in zone 1 and SQL VM 2 in zone 2, joined by an AG. A fire in zone 1's building does not stop the database.

### Azure Site Recovery (ASR)
- **Replicates whole VMs** (disks) to another Azure region, or from on-premises to Azure.
- **Not SQL-aware.** It does not understand transactions. RTO and RPO are at VM level.
- Gives **crash-consistent** recovery points every 5 minutes. **App-consistent** points use **VSS** (Volume Shadow Copy Service, a Windows feature that briefly freezes disk writes for a clean snapshot), which can affect SQL Server if not tuned.
- Can roll a VM back to a point before a **ransomware** infection.
- **Multi-VM consistency** recovers several VMs to the same point, but adds overhead. Use it only when you need it.
- After failover, the new VMs are **not protected until you re-enable replication**.
- Good when you want simple DR for many VMs, or SQL-level DR is too complex. For mission-critical SQL, prefer AGs.

## Windows Server Failover Cluster (WSFC) on Azure
Both FCIs and AGs on Windows sit on a **WSFC** (a group of Windows servers that watch each other and move work when one fails). Linux uses **Pacemaker** instead.

| Term | Meaning |
|---|---|
| **Quorum** | A vote that decides if the cluster may keep running |
| **Witness** | An extra vote that breaks ties |
| **Cloud witness** | A witness stored in Azure Storage. **Recommended** in Azure and hybrid |
| **AD DS and DNS** | Needed for a normal cluster. A **workgroup cluster** skips AD DS but still needs DNS |

Azure-specific points:
- Use a **cloud witness** (Windows Server 2016+) when you can. Use a **disk witness** when the FCI uses **Azure shared disks**. A file share witness is the fallback.
- IP addresses are set **in Azure**, not inside the VM. The VMs see them as DHCP addresses.
- Usually **one network card** per VM.
- **Create the cluster with PowerShell**, not the GUI, to handle Azure networking.

```powershell
Install-WindowsFeature Failover-Clustering -IncludeManagementTools   # on each node
Test-Cluster                                                           # validate first
New-Cluster -Name MyWSFC -Node Node1,Node2 -StaticAddress 10.0.0.5 -NoStorage
# Workgroup cluster (no AD DS): add -AdministrativeAccessPoint DNS
```

- From **Windows Server 2019**, Azure clusters use a **DNN** (distributed network name, a cluster name that points at all node IPs, so no load balancer is needed) by default instead of a **VNN** (virtual network name, one floating IP).

## Always On failover cluster instance (FCI)
- Protects the **whole SQL Server instance**: all databases, logins, Agent jobs, linked servers.
- **One copy of the data** on **shared storage**. The nodes take turns owning it.
- Failover **restarts SQL Server** on another node. Users are disconnected briefly. Slower than an AG failover.
- Clients connect to a **virtual name**, so they don't care which node is active.
- Needs **AD DS and DNS**.
- **Shared storage options on Azure:** **Azure shared disks**, **Premium file shares**, **Storage Spaces Direct (S2D)**, or **Azure Elastic SAN** (SQL Server 2022+).

How clients find the active node:
- **Multiple subnets** (each node in its own subnet): **recommended**. No load balancer and no DNN needed.
- **Single subnet**: use a **DNN** (SQL Server 2019 CU2+, Windows Server 2016+, recommended) or a **VNN + Standard Azure Load Balancer**.
- Note: Basic Load Balancer was retired on 30 Sep 2025. Use Standard.

Test an FCI: Failover Cluster Manager → Roles → **Move** → Best Possible Node or Select Node. Or `Move-ClusterGroup`. For shared disks, check each node can take the disks (Storage → Disks → Move Available Storage).

**Example:** A factory runs an old app that needs SQL Agent jobs and DTC (Distributed Transaction Coordinator). It uses a two-node FCI on Azure shared disks. A node failure moves the whole instance with no app change.

## Always On availability groups (AG)
- Protects **one or more databases**, not the whole instance.
- **No shared storage.** Each replica has its own full copy, so storage cost is higher.
- **Faster failover** than an FCI.
- **Readable secondaries** for reports, and backups can run on secondaries.
- **Synchronous commit** (primary waits for the secondary, no data loss, use nearby) or **asynchronous commit** (no wait, possible data loss, use far away).
- **Editions:** Standard = **basic AG**, 1 database and 2 replicas. Enterprise = many databases, up to **9 replicas**.
- Instance-level objects (logins, jobs) are **not** copied. **Contained AGs** (SQL Server 2022+) fix this: each has its own `master` and `msdb`, so logins, permissions and Agent jobs fail over with the AG.

### Configure an AG on Azure VMs
1. Build the WSFC.
2. Enable the AG feature (SQL Server Configuration Manager or PowerShell), then **restart** SQL Server.
3. Create the AG with SSMS, T-SQL or PowerShell.
4. Create the **listener** (one name apps use to reach the current primary).

```powershell
Enable-SqlAlwaysOn -ServerInstance "SQLVM1"
```

Each replica has two key settings:
| Setting | Choices | Meaning |
|---|---|---|
| **Availability mode** | `SYNCHRONOUS_COMMIT` / `ASYNCHRONOUS_COMMIT` | Wait for the secondary to harden the log, or not |
| **Failover mode** | `AUTOMATIC` / `MANUAL` | Automatic failover needs **synchronous commit** on both sides |

Short T-SQL example (run on the primary):
```sql
CREATE AVAILABILITY GROUP [AG1]
FOR DATABASE [SalesDb]
REPLICA ON
 'SQLVM1' WITH (ENDPOINT_URL = 'TCP://SQLVM1.contoso.com:5022',
   AVAILABILITY_MODE = SYNCHRONOUS_COMMIT, FAILOVER_MODE = AUTOMATIC),
 'SQLVM2' WITH (ENDPOINT_URL = 'TCP://SQLVM2.contoso.com:5022',
   AVAILABILITY_MODE = SYNCHRONOUS_COMMIT, FAILOVER_MODE = AUTOMATIC,
   SECONDARY_ROLE (ALLOW_CONNECTIONS = READ_ONLY));
-- Then on SQLVM2: ALTER AVAILABILITY GROUP [AG1] JOIN;
```
- The database must use the **FULL** recovery model and have a **full backup** before it can join.
- **Port 5022** is the default **database mirroring endpoint** that replicas use to talk. Open it in NSGs and the Windows firewall.
- **Automatic seeding** (`SEEDING_MODE = AUTOMATIC`) copies the database to the secondary for you. Otherwise you back up and restore it `WITH NORECOVERY` yourself.

Listener options in Azure:
- **Multiple subnets** (replicas in different subnets): **recommended**. The listener has one IP per subnet. No load balancer.
- **DNN listener** (SQL Server 2016 SP3 / 2017 CU25 / 2019 CU8+, Windows Server 2016+).
- **VNN listener in one subnet**: needs a **Standard internal load balancer** with **Floating IP** (Direct Server Return) on, and a **probe port** (such as **59999**) set on the listener IP resource.

```powershell
Get-ClusterResource "AG1_IP" | Set-ClusterParameter ProbePort 59999
Test-NetConnection AG1-Listener -Port 1433   # check the listener answers
```

**Example:** An online shop puts two sync replicas in two zones (HA, automatic failover) and one async replica in another region (DR). Reports run on the readable secondary.

## Distributed availability groups
- An **"AG of AGs"**: links two separate AGs, each on its **own cluster** with its own quorum and witness.
- Added in SQL Server 2016 (Enterprise).
- Good for **multi-region DR** and **on-premises to Azure**. No single cluster stretched across regions.
- Each side manages itself, and **failback** is simpler.
- In Azure, each region needs its own load balancer for the endpoint port **5022** (when using VNN).

## AG stretched across regions or hybrid
- One AG with replicas in two regions, or on-premises plus Azure. One feature gives HA and DR.
- Works in Standard or Enterprise.
- Needs good network, **AD DS and DNS in every site**, and careful **witness placement** to avoid **split brain** (two nodes both thinking they are primary).

## Log shipping
- Protects **databases**. Takes **transaction log backups** on the primary, **copies** them to a secondary, and **restores** them there on a schedule.
- Creates a **warm standby** (a copy that is minutes behind and ready to bring online).
- **Manual failover only.** Some data loss is expected unless the failover is planned.
- **Simple, cheap, reliable.** Works even on slow or unreliable networks.
- Can protect an **FCI** at a DR site, because an FCI does not copy data by itself.
- Needs the **FULL** or **BULK_LOGGED** recovery model.
- Set up with the **SSMS wizard** (database Properties → Transaction Log Shipping) or **T-SQL**. Three SQL Agent jobs do the work: **backup** (on primary), **copy** and **restore** (on secondary). An optional **monitor server** records history and raises alerts.
- Take any extra log backups with **COPY_ONLY**, or you break the chain.

Steps to configure:
1. Set the database to **FULL** recovery and take a full backup.
2. Create a **shared folder** for the log backups that the secondary can read.
3. Start the wizard (or T-SQL) on the primary. Set the backup schedule and the backup folder.
4. Add the secondary server. Choose how to initialise it (new full backup, existing backup, or already restored).
5. Choose the restore mode on the secondary:
   - **No recovery mode**: the database can't be read.
   - **Standby mode**: read-only between restores. Users are disconnected during each restore.
6. Optionally add a **monitor server**.

The T-SQL stored procedures start with `sp_add_log_shipping_` (for example `sp_add_log_shipping_primary_database` on the primary and `sp_add_log_shipping_secondary_database` on the secondary).

To **fail over**: take a **tail-log backup** of the primary if it is still reachable (`BACKUP LOG ... WITH NORECOVERY`), copy and restore all remaining logs on the secondary, then restore the last one `WITH RECOVERY`. Repoint the apps, because log shipping has **no listener**.

**Example:** A news site ships logs every 15 minutes to a VM in another region. If the main region fails, the DBA restores the last logs and brings the standby online.

## FCI vs AG vs log shipping
| | FCI | AG | Log shipping |
|---|---|---|---|
| Protects | Whole instance | Databases | Databases |
| Shared storage | Required | No | No |
| Data copies | One | One per replica | One per secondary |
| Automatic failover | Yes | Yes (sync replicas) | **No** |
| Readable copy | No | Yes | Only in standby mode, between restores |
| Complexity | High | Medium | Low |
| Cost | Medium-high | High | Low |

## Which VM solution when
| Scenario | Solution |
|---|---|
| HA in one region | AG or FCI |
| DR between Azure regions | AG across regions or distributed AG |
| DR from on-premises to Azure | Distributed AG or log shipping (or MI link to SQL MI) |
| Simple VM-level DR, many VMs | Azure Site Recovery |
| Protect instance-level objects | FCI (or contained AG) |
| Lowest-cost DR, manual work OK | Log shipping |

## Hybrid HA/DR
**Hybrid** means on-premises SQL Server plus Azure.
- Uses for Azure: a **cloud DR site**, a **migration** staging area, **read replicas** for reports, and **cheap backup storage** in Blob Storage.
- Mostly **IaaS**: the usual SQL features (AG, distributed AG, log shipping, FCI with log shipping) work across sites.
- Needs **AD DS, DNS and WSFC** for AG-based designs.
- **Network is the key.** Use **ExpressRoute** (private dedicated link, preferred) or **site-to-site VPN** (cheaper, slower). **Never** expose SQL VMs to the public internet.
- PaaS can't join an on-premises AG or cluster. Exceptions:
  - **Managed Instance link** (lesson 3): DR between SQL Server and SQL MI.
  - **Transactional replication**: on-premises SQL Server (publisher) can push to SQL MI or SQL Database (subscriber). SQL MI can also be a publisher and push back to SQL Server. SQL Database can only be a subscriber.
- Even with no hybrid DR, you can **back up on-premises databases to Azure Blob Storage** (see lesson 4).

**Example:** A manufacturer keeps its primary AG replica in its own datacenter and an async secondary on an Azure VM, linked by ExpressRoute, with a cloud witness.

## Remember for the exam
- Availability set = one datacenter. Zone = across datacenters. Site Recovery = across regions.
- Sets and zones protect the **VM**, not SQL Server. You still need FCI/AG/log shipping.
- ASR is **not SQL-aware**. Pick it for "simple DR for many VMs".
- WSFC on Azure: **cloud witness**, **PowerShell** to create, IPs set **in Azure**.
- FCI = whole instance + **shared storage** (shared disks, Premium file shares, S2D, Elastic SAN).
- Multi-subnet = no load balancer. Single subnet = **DNN** (preferred) or VNN + **Standard** LB with **probe port** and **Floating IP**.
- AG Standard edition = basic AG, 1 DB, 2 replicas. Enterprise = up to 9 replicas.
- Log shipping = **manual failover**, cheapest, works on weak networks.
- Hybrid: **ExpressRoute** preferred, never public internet.

## Quick check
1. Your VMs must survive a whole datacenter outage inside one region. What do you use?
   <details><summary>Answer</summary>Availability zones. Availability sets only protect inside one datacenter.</details>
2. An AG has both replicas in one subnet with a VNN listener. Apps can't reach the listener after failover. What do you check?
   <details><summary>Answer</summary>The load balancer health probe: the probe port on the listener IP resource and Floating IP. Better: use a DNN or multi-subnet design.</details>
3. You need cheap DR to another region and can accept manual failover and a few minutes of data loss. Which feature?
   <details><summary>Answer</summary>Log shipping. It is simple and low cost but has no automatic failover.</details>
