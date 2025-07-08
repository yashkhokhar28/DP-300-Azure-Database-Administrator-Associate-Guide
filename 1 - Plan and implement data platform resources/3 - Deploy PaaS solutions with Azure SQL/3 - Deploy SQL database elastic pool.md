## ✅ **What is an Elastic Pool?**

An **Elastic Pool** is a shared resource container where:

- Multiple **Azure SQL Databases** share the same **compute (CPU), memory, and storage**.
- Databases can dynamically use more or fewer resources depending on demand.
- It’s like an **on-premises SQL Server instance** where multiple databases share the same SQL Server resources.

---

### 🔑 **Key Benefits:**

- **Cost savings:** Instead of allocating separate resources for each database, you pay for a pool of resources shared by all databases.
- **Automatic resource sharing:** When one database experiences a spike in workload, it can borrow unused resources from other databases.
- **Scalability:** You can easily increase or decrease the total resources of the pool.

---

## ✔️ **How to Create and Manage an Elastic Pool**

### 🔨 **Create an Elastic Pool:**

1. In the Azure portal, search for **“SQL elastic pools”**.
2. Select **“Create”**.
3. Fill in:

   - Resource group, region.
   - Pool name, server, and compute/storage configuration.

---

### ➕ **Add a Database to an Existing Pool:**

1. Go to your **Elastic Pool** in the Azure portal.
2. Select **+ Add databases**.
3. Choose databases from the list.
4. Click **Apply**, then **Save**.

---

### 🔍 **Monitor and Manage Pool Resources:**

- Azure Portal shows:

  - Which databases use the most resources.
  - Total resource usage (CPU, storage).
  - Health of the pool.

- **Resize the pool:**

  - Use the **Configure** option.
  - Adjust DTUs (if using DTU model), vCores (if using vCore model), and storage size.
  - Set **min/max limits** for each database in the pool.
  - Add or remove databases.
  - Increase or decrease the overall pool size.

  Some changes happen **online**, meaning no downtime. But resizing the pool may **briefly disconnect active connections.**

---

## 🔧 **Use Case Example:**

Elastic pools are **ideal for SaaS apps** where:

- Each tenant has its own database.
- Not all tenants use the database equally.
- Example: Some customers are active only during business hours, others during the night.

---

## ⚠️ **Key Points to Watch:**

- If **one database consumes too much**, it may starve others.
- Monitor regularly and adjust **min/max resources** to ensure fair sharing.

---
