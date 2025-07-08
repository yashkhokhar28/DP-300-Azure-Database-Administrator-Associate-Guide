## 🔍 **1. Azure Migrate (Lift and Shift)**

- Moves the **whole physical or virtual machine** (OS + SQL Server) as-is to Azure VM.
- No changes to SQL Server version or operating system.
- Syncs data during the migration, so **the source server stays online until cutover.**
- Minimal downtime during final switch-over.

---

## 🔁 **2. Transactional Replication**

Used if **Always On** isn’t available and you want to **minimize downtime.**

➡️ **Steps:**

- On-premises SQL Server = **Publisher**
- Azure VM SQL Server = **Subscriber**

🔧 **Process:**

1. **Set up a Distributor** (manages replication)
2. **Create a Publication** (select tables/articles to replicate)
3. **Create a Subscription** on the Azure VM.

✔ Once Azure has the latest data, point your applications to the Azure VM and remove replication.

---

## 🔀 **3. Log Shipping**

- Continuously ships **transaction log backups** from on-premises SQL Server to Azure VM.
- Source remains active until you cut over.
- Less complex than Always On but still minimizes downtime.

🔧 Steps:

- Backup transaction logs → copy to Azure VM → restore them.
- Do a final log backup + restore → cut over.

---

## 🔄 **4. Detach and Attach from a URL**

- **Offline method.**
- Detach DB → move database (`.mdf`) and log (`.ldf`) files to Azure Blob Storage.
- Attach the database from Blob Storage on Azure VM.

Good for **larger databases** when downtime is acceptable.

---

## 📦 **5. Convert to VM and Deploy as New VM**

- Convert the whole on-premises SQL Server VM to a virtual hard disk (**VHD**).
- Upload to Azure → Deploy it as a new Azure VM.

Used when:

- Migrating **older SQL Server versions**
- Moving **system + user databases** together.
- Bringing your own SQL Server license (BYOL).

---

## 🧳 **6. Import/Export Wizard / BACPAC**

- **.bacpac** files contain schema + data.
- Export on-premises database to a bacpac → import it on Azure VM.
- Works well for **small or simple databases.**

For larger production environments, use **SqlPackage** with parallel imports for better performance.

---

## 🔗 **7. SQL Server Data Files on Azure Storage**

- Hybrid approach, not a full migration.
- Store SQL Server data files (mdf/ldf) on Azure Storage but still run SQL Server on-premises.
- Good for **gradual migration** and shared storage solutions.

---

## 📤 **8. Bulk Copy (bcp)**

- Command-line tool to **export/import data at the table level.**
- Efficient for **migrating large tables**, but you need to manage schema, indexes, and constraints manually.

---

## 🔄 **9. Azure Data Factory**

- Used for **data-level migration and transformation.**
- Transfers data between SQL Server and Azure SQL/VM.
- Ideal when moving data **from multiple sources** or for **BI (business intelligence) workloads.**

---

## 📦 **10. Windows Import/Export Service**

- Ship your **physical hard drives** to an Azure data center.
- Used when **network transfer is too slow** for large datasets.

---

## ✅ **Summary of Use Cases**

| Method                          | Downtime | Use Case                                            |
| ------------------------------- | -------- | --------------------------------------------------- |
| Azure Migrate                   | Low      | Full server lift-and-shift                          |
| Transactional Replication       | Low      | No Always On, continuous sync                       |
| Log Shipping                    | Low      | Simple backup-based sync                            |
| Detach & Attach                 | High     | Large database, offline window acceptable           |
| Convert to VM                   | Medium   | Older SQL versions, move system + user DBs together |
| Import/Export Wizard / BACPAC   | Medium   | Smaller DBs, schema + data only                     |
| SQL Data Files on Azure Storage | None     | Hybrid storage, gradual cloud move                  |
| bcp                             | High     | Table-level migrations, ETL tasks                   |
| Azure Data Factory              | Low/Med  | Data sync + transform pipelines, BI workloads       |
| Windows Import/Export Service   | None     | Large datasets, offline transfer via hard drives    |

---
