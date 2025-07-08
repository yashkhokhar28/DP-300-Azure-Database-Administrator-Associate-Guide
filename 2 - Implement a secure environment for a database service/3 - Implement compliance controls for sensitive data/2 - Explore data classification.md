## ✅ **What Is Data Classification?**

Data classification is the process of **tagging database columns** with labels indicating how sensitive the data is.
Example:

- **Public** – Non-sensitive data
- **Confidential** – Sensitive data (e.g., email, phone numbers)
- **Highly Confidential** – Critical data (e.g., credit card numbers, personal ID numbers)

This tagging helps organizations:

- Understand which data is sensitive.
- Meet compliance and privacy requirements.
- Make informed decisions about applying **encryption**, **auditing**, or **access control**.

---

## 🔍 **Where Is the Classification Stored?**

Earlier (pre-SQL Server 2019), classification was stored as **extended properties** of the column.
From **SQL Server 2019+** and in **Azure SQL Database/Managed Instance**, it is stored in a built-in system view:
➡️ **`sys.sensitivity_classifications`**

---

## 🛡️ **How to Classify Data**

| Method                      | Tool                                                |
| --------------------------- | --------------------------------------------------- |
| **Auto-scan & suggestions** | Azure Portal or SQL Server Management Studio (SSMS) |
| **Manual classification**   | Azure Portal, SSMS, or T-SQL                        |
| **T-SQL example:**          |                                                     |
|                             | \`\`\`sql                                           |
|                             | ADD SENSITIVITY CLASSIFICATION TO                   |
|                             | \[Application].\[People].\[EmailAddress]            |
|                             | WITH (LABEL='PII', INFORMATION_TYPE='Email')        |
|                             | \`\`\`                                              |

The classification engine **scans for column names** like "email", "dob", "ssn", and suggests sensitivity labels. But if your columns are poorly named (like "col1"), the engine won't identify them as sensitive.

---

## 🛡️ **Benefits of Data Classification**

- **Security Awareness:** Helps your teams know where the sensitive data lives.
- **Audit Readiness:** Simplifies compliance reporting.
- **Encryption Planning:** Shows which columns need encryption (e.g., Always Encrypted).
- **Access Control:** Helps DBAs and developers apply fine-grained access restrictions.

---

## ⚙️ **Customizing Sensitivity Labels**

If the built-in labels don’t meet your needs, you can:

- Create custom sensitivity labels and information types.
- Manage them through **Microsoft Defender for Cloud > Data Discovery & Classification**.
- Example custom labels: "Internal Use", "Health Records", "Financial Data".

You need **root management group admin rights** in Azure to create/manage these labels.

---

## 📊 **Example Process in Azure Portal:**

1. Open Azure SQL Database.
2. Go to **Security > Data Discovery & Classification**.
3. See the **classification recommendations**.
4. Accept or adjust them.
5. Save your classification labels.

---

## ✔️ **Summary**

- Data classification is a foundational step in protecting sensitive data.
- It identifies and labels columns in your database.
- It enables better compliance, auditing, and security decision-making.

---
