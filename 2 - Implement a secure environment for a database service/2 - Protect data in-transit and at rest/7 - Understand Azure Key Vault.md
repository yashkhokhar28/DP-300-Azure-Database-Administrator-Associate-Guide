## 🔐 **What is Azure Key Vault?**

Azure Key Vault is a **secure cloud service** for storing and managing **secrets**, such as:

- Passwords
- Encryption keys
- Certificates
- API keys

It provides a **centralized, secure way** to store and control access to sensitive data for applications and services.

---

## 🔍 **Why Use Key Vault?**

| Benefit                        | Description                                                                              |
| ------------------------------ | ---------------------------------------------------------------------------------------- |
| Centralized secret management  | Manage all your application secrets in one place                                         |
| Strong access controls         | Uses its own **Role-Based Access Control (RBAC)** separate from Azure subscription roles |
| Secure storage                 | Keys are software-protected (**Standard** tier) or **HSM-protected** (**Premium** tier / Managed HSM) 🔄 |
| Simplifies encryption features | Works seamlessly with Azure services like SQL Server for encryption                      |
| Auditing and monitoring        | Track who accessed or modified secrets                                                   |

---

## 🔑 **SQL Server Integration with Azure Key Vault**

### What can SQL Server store in Key Vault?

- **Transparent Data Encryption (TDE) certificates** (encrypts your database files)
- **Backup encryption certificates**
- **Always Encrypted column master keys** (protects specific data columns)

This works for:

- SQL Server running **on Azure Virtual Machines**
- SQL Server running **on-premises** (more complex setup)

---

## ⚙️ **How to Set Up Azure Key Vault for SQL Server**

### 1. **Key Vault Setup**

- Create a **Key Vault** in Azure.
- Add an **application registration** (App ID & secret).
- Grant the app the **Key Vault Crypto Service Encryption User** RBAC role (recommended) or legacy access-policy permissions (**get, list, wrapKey, unwrapKey**). 🔄
- SQL Server **2022 CU17+ on an Azure VM** (or SQL Server 2025 enabled by Azure Arc) can use a **managed identity** instead of an app registration. On-premises SQL Server still needs a service principal. 🔄

### 2. **Configure SQL Server Credentials**

On SQL Server (inside the Azure VM):

- Create a **login** (for example, `sql_keyvault_user`).
- Create a **credential** with:

  - Identity: The **Key Vault name**.
  - Secret: The **App ID (Client ID)** and **App Secret**.

Example:

```sql
-- Prereqs: install the SQL Server Connector for Azure Key Vault, sp_configure 'EKM provider enabled', 1,
-- then CREATE CRYPTOGRAPHIC PROVIDER AzureKeyVault_EKM FROM FILE = '...\Microsoft.AzureKeyVaultService.EKM.dll'  -- 🔄
CREATE CREDENTIAL sysadmin_ekm_cred
WITH IDENTITY = 'mykeyvault',                          -- key vault NAME  -- 🔄
SECRET = '<ClientIdWithoutHyphens><ClientSecret>'     -- app ID + secret concatenated  -- 🔄
FOR CRYPTOGRAPHIC PROVIDER AzureKeyVault_EKM;          -- 🔄
```

### 3. **Create an Asymmetric Key**

- First, create the key in Azure Key Vault.
- Then, in SQL Server:

```sql
CREATE ASYMMETRIC KEY MyKey
FROM PROVIDER [AzureKeyVault_EKM]                     -- 🔄
WITH PROVIDER_KEY_NAME = 'MyAzureKey',
     CREATION_DISPOSITION = OPEN_EXISTING;            -- 🔄
```

Now, SQL Server can use this key for:

- Transparent Data Encryption (TDE)
- Backup encryption
- Column-level (cell) encryption. Always Encrypted CMKs in Key Vault are used by the **client driver**, not through SQL Server EKM (Learn lists Always Encrypted here). 🔄

---

## 🛡️ **Key Point About Security**

Even if someone is an Azure **Subscription Admin**, they **won’t have access to Key Vault** unless given permission. This isolates database security from general Azure admin roles.

---

## 🔗 **When is This Useful?**

| Scenario                                | Why it Helps                                          |
| --------------------------------------- | ----------------------------------------------------- |
| Securing sensitive data in the cloud    | Prevents unauthorized access to encryption keys       |
| Enabling compliance (GDPR, HIPAA, etc.) | Centralized, auditable key management                 |
| Automating secure database deployments  | Keys and certificates provisioned at VM creation time |
| Protecting on-prem SQL Server           | Can also configure, but setup is more manual          |

---
