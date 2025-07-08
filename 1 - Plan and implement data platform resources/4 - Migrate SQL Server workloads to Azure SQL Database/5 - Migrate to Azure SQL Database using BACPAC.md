## 📦 **What is a BACPAC File?**

- A **BACPAC file** is a **compressed archive** containing:

  - **Schema**: Tables, views, stored procedures, etc.
  - **Data**: All the rows from your tables.

- BACPAC files are useful for **moving or archiving databases**, especially when migrating **from on-premises SQL Server to Azure SQL Database**.

---

## 🔄 **How Does a BACPAC Migration Work?**

1. **Export**: Create a `.bacpac` file from the source SQL Server database. This can be done using **SQL Server Management Studio (SSMS)** or **SqlPackage.exe**.
2. **Store**: Upload the `.bacpac` file to:

   - **Azure Blob Storage** (recommended for Azure migrations)
   - or keep it in **local storage**.

3. **Import**: Use the **Azure portal** or the **SqlPackage tool** to import the `.bacpac` file into Azure SQL Database.

---

## ✅ **Steps to Import a BACPAC File via the Azure Portal**

### 1. Go to the Azure SQL Server page

- In the Azure portal, navigate to your **Azure SQL logical server**.

### 2. Select **“Import database”**

- This will prompt you to select the source BACPAC file.

### 3. Choose the storage account and the BACPAC file

- Locate your `.bacpac` file in the Azure Storage container.

### 4. Configure your new database

- Set:

  - The **new database name**
  - **Compute and storage size**
  - **Authentication credentials**

### 5. Start the import

- Monitor progress under **Import/Export History**.
- When the import completes successfully, the database will be ready for use.

---

## ⚙️ **Faster Import Using SqlPackage.exe**

The **SqlPackage.exe** utility is a **command-line tool** that’s faster than the Azure Portal for importing large BACPAC files.

Example command:

```bash
SqlPackage.exe /a:import \
/tcs:"Data Source=<server-name>.database.windows.net;Initial Catalog=myMigratedDatabase;User Id=<your_server_admin_account_user_id>;Password=<your_server_admin_account_password>" \
/sf:AdventureWorks2019.bacpac \
/p:DatabaseEdition=Premium \
/p:DatabaseServiceObjective=P6
```

### Options explained:

| Option                         | Meaning                                               |
| ------------------------------ | ----------------------------------------------------- |
| `/a:import`                    | Action: Import                                        |
| `/tcs:`                        | Target connection string                              |
| `/sf:`                         | Source file (the BACPAC)                              |
| `/p:DatabaseEdition=`          | SQL Database edition (e.g., Basic, Standard, Premium) |
| `/p:DatabaseServiceObjective=` | Performance level (e.g., P6)                          |

---

## ⚡ **Performance Tip**

- Scale your Azure SQL Database **up to a higher tier** (e.g., Premium P6) before starting the import for faster processing.
- After migration, **scale down** to save costs.

---

## 🔍 **Summary of Benefits**

✔️ Simple migration method
✔️ Works well for **small/medium databases**
✔️ Can be scripted or done through the portal
✔️ Useful for **initial migrations or backups**

---

## ⚠️ **Limitations**

- BACPAC only works for **offline migrations** (downtime required).
- **Logins, jobs, and linked servers** are **NOT migrated** (because BACPAC contains only database-level objects).
- **Large databases (>200 GB)** may take longer or fail due to storage or timeout limitations.

---
