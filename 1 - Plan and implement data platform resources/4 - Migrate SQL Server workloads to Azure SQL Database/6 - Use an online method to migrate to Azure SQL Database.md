## ✅ **Why Use Transactional Replication for Migration?**

- If your application **can’t afford downtime**, replication allows you to:

  - Keep your **source database online**.
  - **Continuously sync changes** from the source SQL Server to Azure SQL Database.

- Example: A **warehouse inventory database** where operations run 24/7.

---

## 🔑 **What is Transactional Replication?**

Transactional Replication works by:

1. **Taking a snapshot** of the selected tables in the source database (initial load).
2. **Continuously tracking data changes** using the SQL Server transaction log.
3. **Pushing those changes** to Azure SQL Database almost in real-time.

Azure SQL Database supports **Push Subscriptions**, meaning the source database actively sends changes to the destination.

---

## 🔧 **Key Replication Components**

| Component        | Description                                                                                                   |
| ---------------- | ------------------------------------------------------------------------------------------------------------- |
| **Publisher**    | The source SQL Server database.                                                                               |
| **Subscriber**   | Azure SQL Database, the destination.                                                                          |
| **Distributor**  | Manages the flow of data from Publisher to Subscriber (can be on the Publisher itself or another SQL Server). |
| **Article**      | Individual database object to replicate (e.g., a table).                                                      |
| **Publication**  | A collection of articles.                                                                                     |
| **Subscription** | A request from a Subscriber to receive data from a Publication.                                               |

---

## 🔨 **Steps to Set Up Replication**

### 1. **Set Up the Distributor**

- Use `sp_adddistributor` and `sp_adddistributiondb` to create and configure the distribution database.

### 2. **Enable the Publisher**

- Enable the source database (`AdventureWorks` in this example) for replication.

### 3. **Create the Publication**

- Use `sp_addpublication` to define the publication (`REPL-AdventureWorks`).
- This step sets parameters like:

  - Whether snapshot files are compressed.
  - Continuous sync mode.

### 4. **Add Articles (Tables to Replicate)**

- Use `sp_addarticle` to add the `[Person].[Person]` table to the publication.

### 5. **Create a Subscription**

- Use `sp_addsubscription` to set up Azure SQL Database as the **subscriber**.
- **Note:** Azure SQL Database must exist before running this.

### 6. **Configure the Push Subscription Agent**

- Use `sp_addpushsubscription_agent` to create an agent that pushes data from the publisher to the subscriber.

---

## 🛡️ **Authentication & Connection Notes**

- Azure SQL Database supports only **SQL Authentication** for replication.
- You must create SQL users with sufficient privileges on both the source and Azure SQL Database.

---

## ▶️ **Start Replication & Monitor**

After setting up the components, start the agents:

- **Snapshot Agent**: Creates the initial snapshot of the data.
- **Log Reader Agent**: Monitors the source database for changes.
- **Distribution Agent**: Pushes changes to Azure SQL Database.

You monitor replication from **SQL Server Management Studio (SSMS)** on the publisher:

- View agent status.
- Check synchronization.

---

## 🔍 **Monitoring Example Screens**

- You will see status like **"Running"**, **"Success"**, or **"Failure"** for:

  - Snapshot Agent
  - Log Reader Agent
  - Synchronization status of the subscription.

---

## 🔚 **Cutover Process**

Once all data is replicated:

1. Stop your application writes to the source.
2. Make sure **all changes have been synced**.
3. Point your application to the **Azure SQL Database**.
4. Remove the replication setup.

---

## ⚠️ **Important Limitations**

- Only **Transactional Replication** works online to Azure SQL Database (no merge replication, peer-to-peer, etc.).
- Schema changes during replication are limited.
- Requires careful configuration of SQL Server Agent jobs.
- Replication monitoring is **NOT available in the Azure portal**, only in SQL Server Management Studio.

---

## ✅ **Use Case Summary**

| Scenario                                | Suitable?                       |
| --------------------------------------- | ------------------------------- |
| 24/7 mission-critical workloads         | ✔️ Yes                          |
| Small to medium-sized databases         | ✔️ Yes                          |
| Large schema changes during migration   | ❌ No                           |
| Onetime migration with downtime allowed | ❌ No, better use BACPAC or DMS |

---
