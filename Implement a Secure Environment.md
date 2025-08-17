# Implement a Secure Environment

This section of the DP-300 exam covers securing Azure SQL Database, Azure SQL Managed Instance, and SQL Server environments. It includes configuring authentication and authorization, implementing encryption for data at rest and in transit, and applying compliance controls for sensitive data. Key principles like least privilege are emphasized to minimize risks. You'll learn to troubleshoot issues, manage via T-SQL (Transact-SQL), and use features like audits, masking, and ledger for regulatory compliance.

Real-world context: In a healthcare organization handling patient records, implementing these security measures ensures compliance with regulations like HIPAA (Health Insurance Portability and Accountability Act). For instance, using Microsoft Entra ID (formerly Azure Active Directory) for authentication prevents unauthorized access, while Transparent Data Encryption (TDE) protects data at rest from breaches, and Dynamic Data Masking hides sensitive details like social security numbers from non-admin users during queries.

## 2.1 Configure Database Authentication and Authorization

This subsection focuses on setting up secure access using Microsoft Entra ID, managing principals, users, permissions, and troubleshooting, all while adhering to least privilege.

### Configure Microsoft Entra Authentication for Azure SQL Database, Azure SQL Managed Instance, and SQL Server

Microsoft Entra ID (formerly Azure Active Directory) provides centralized identity management for secure authentication across Azure SQL services.

- **For Azure SQL Database**: Select the directory containing your SQL resource as the current directory in the Azure portal. Search for SQL servers, open the logical server pane, go to Settings > Microsoft Entra ID, set an admin, and save. This enables multi-factor authentication (MFA) and role-based access control (RBAC).

- **For Azure SQL Managed Instance**: Grant permissions by adding managed identity to roles and app role permissions. Enable outbound communication and Microsoft Entra authentication to support hybrid identities.

- **For SQL Server**: Create and register a Microsoft Entra application, grant permissions, create/assign certificates, configure via Azure portal, create logins/users, and connect using supported methods like SSMS (SQL Server Management Studio). Parameters are stored in a secure location. Use authentication examples in SSMS for testing.

Real-world example: A global bank configures Microsoft Entra authentication for Azure SQL Database to integrate with their corporate directory. This allows employees to log in with single sign-on (SSO), reducing password fatigue and enhancing security during audits, where access logs show who queried financial data.

### Configure Security Principals

Security principals are entities (users, groups, applications) that can be authenticated and authorized. Service principals ensure proper security for applications in single- or multi-tenant scenarios.

- **Service Principal Authentication**: Supports client certificates or secrets. Find accounts using Azure CLI (Command-Line Interface) or PowerShell (e.g., Get-AzureADServicePrincipal). Assess privileges, evaluate credential storage, and move from user accounts to service principals with just enough permissions.

Real-world example: In a multi-tenant SaaS (Software as a Service) application for project management, configure service principals to allow tenant-specific apps to access databases without over-privileging, ensuring one tenant's data isn't exposed to another during API calls.

### Create Users from Microsoft Entra Identities

Map Microsoft Entra identities to database users for seamless access.

- **Process**: In the master database, use T-SQL like `CREATE LOGIN [nativeuser@aadsqlmi.onmicrosoft.com] FROM EXTERNAL PROVIDER` to create a server principal (login).

Real-world example: An e-commerce platform creates users from Microsoft Entra identities for developers, granting them access to staging databases. This integrates with Azure DevOps, allowing automated deployments without storing credentials in code repositories.

### Configure Database and Object-Level Permissions Using Graphical Tools

Grant, revoke, or deny permissions on databases, tables, views, functions, and stored procedures via tools like SSMS.

- **Table and View Permissions**: Include SELECT, INSERT, UPDATE, DELETE, CONTROL, REFERENCES, TAKE OWNERSHIP, VIEW CHANGE TRACKING, VIEW DEFINITION.

- **Function and Stored Procedure Permissions**: Include ALTER, CONTROL, EXECUTE, VIEW CHANGE TRACKING, VIEW DEFINITION.

Real-world example: In a CRM (Customer Relationship Management) system, use SSMS to grant SELECT on customer views to sales teams but deny UPDATE, preventing accidental data changes while allowing read access for reporting.

### Apply the Principle of Least Privilege for All Securables

Grant only necessary permissions to reduce attack surface.

- **Data Access Control**: Use identity platform's consent framework during app development to request minimal access.

- **For Deployed Apps**: Periodically review granted permissions to mitigate risks. Fully understand least-privileged permission sets to avoid overprivileged apps.

Real-world example: A logistics company applies least privilege by granting a reporting app only SELECT on shipment tables, reviewed quarterly. This prevented a potential breach when a compromised app couldn't delete data during a phishing attack.

### Troubleshoot Authentication and Authorization Issues

Common issues include transient faults, login failures, resource limits, and connection strings.

- **Transient Faults/Login Failures**: Check Azure Service Dashboard for outages; monitor resource limits; file support requests for extended failures.

- **Disabled Logins**: Use sys.sql_logins to check status; enable with ALTER LOGIN.

- **Missing Access**: Create users with CREATE USER and grant access.

- **Connection Strings**: Edit in Azure portal to include passwords/server names; ensure client apps reference them correctly.

Real-world example: During a cloud migration for a retail chain, troubleshoot login failures by checking sys.sql_logins and enabling disabled accounts, resolving issues caused by resource spikes during Black Friday sales.

### Manage Authentication and Authorization by Using T-SQL

Use T-SQL for credentials and permissions.

- **Server-Level Credentials**: Created with CREATE CREDENTIAL; used for multiple databases.

- **Database-Scoped Credentials**: Created with CREATE DATABASE SCOPED CREDENTIAL; enhances portability; not mapped to logins/users.

Real-world example: In an automated ETL (Extract, Transform, Load) process for analytics, use database-scoped credentials to connect to external sources, making the database movable between instances without recreating server-level logins.

## 2.2 Implement Security for Data at Rest and Data in Transit

Protect data when stored (at rest) or moving (in transit) using encryption and network controls.

### Implement Transparent Data Encryption (TDE)

TDE encrypts the entire database transparently without app changes.

- **Architecture**: Service Master Key (created at SQL setup) encrypts Database Master Key (CREATE MASTER KEY). This creates a certificate (CREATE CERTIFICATE) that encrypts the Database Encryption Key (DEK, CREATE DATABASE ENCRYPTION KEY). Enable with ALTER DATABASE SET ENCRYPTION ON.

Real-world example: A financial institution enables TDE on customer transaction databases to protect against physical theft of storage media, ensuring compliance with PCI DSS (Payment Card Industry Data Security Standard) without modifying legacy apps.

### Implement Object-Level Encryption

Encrypt specific columns or objects using keys.

- **Process**: Create symmetric keys (e.g., for credit card numbers) and encrypt data in tables.

Real-world example: Encrypt salary columns in an HR database with object-level encryption, allowing only payroll admins to decrypt, preventing exposure in case of a general data dump.

### Configure Server- and Database-Level Firewall Rules

Control network access to prevent unauthorized connections.

- **Server-Level Rules**: Allow access to all databases; configured with sp_set_firewall_rule.

- **Database-Level Rules**: Specific to one database; configured with sp_set_database_firewall_rule.

Real-world example: A university configures database-level firewalls to allow student apps access only to enrollment DBs, blocking external IPs and reducing DDoS (Distributed Denial of Service) risks.

### Implement Always Encrypted

Client-side encryption ensures sensitive data never appears plaintext in the database.

- **Features**: Encrypts inside client apps; keys stored in Windows certificate store or Azure Key Vault. Scenarios: On-premises client with Azure data, fully on-premises, or fully Azure.

Real-world example: A telemedicine app uses Always Encrypted to protect patient health data during transit to Azure SQL, ensuring even DB admins can't view plaintext, complying with GDPR (General Data Protection Regulation).

### Implement Always Encrypted with VBS Enclaves

Virtual Secure Mode (VSM) enclaves provide software-based secure computation without special hardware.

- **Enable for New Database/Pool**: In Azure portal, create database/pool, set Enable secure enclaves to ON in Security tab.

- **Enable for Existing**: For database, go to Data Encryption > Always Encrypted > set ON. For pool, Configuration > Always Encrypted > set ON.

Real-world example: An insurance firm enables VBS enclaves for querying encrypted claims data, allowing computations like aggregations without decrypting, enhancing privacy in multi-user environments.

### Configure Private Links and Service Endpoints

Secure connections and user access.

- **Adding Users**: SQL Authentication: Create logins/users with usernames/passwords stored in SQL. Microsoft Entra: Contained users mapped to directory identities, no master login.

- **Secure Connection Strings**: Request encrypted connections; don't trust server certificates.

Real-world example: Configure private links for a corporate intranet DB to route traffic over Azure backbone, avoiding public internet, and use contained Microsoft Entra users for seamless employee access without VPNs.

## 2.3 Implement Compliance Controls for Sensitive Data

Apply classifications, audits, tracking, masking, ledger, and row-level security for data governance.

### Apply a Data Classification Strategy

Classify data as Public, General, Confidential, Highly Confidential to guide protection.

- **Steps**: Discovery/recommendations, labeling, visibility.

- **Methods**: Use SSMS, Azure portal, or T-SQL (ADD SENSITIVITY CLASSIFICATION) for columns like BirthDate (Confidential - GDPR).

Real-world example: A marketing firm classifies email addresses as Confidential, using SSMS to label and report on sensitive data, aiding in breach notifications under CCPA (California Consumer Privacy Act).

### Configure Server and Database Audits

Track events for compliance and insights.

- **Features**: Logs to blob storage; analyze reports for discrepancies, suspicious activity, trends.

Real-world example: Enable audits on a banking DB to log all queries, helping detect insider threats by reviewing trails of unusual access patterns during fraud investigations.

### Implement Change Data Tracking

Track changes for auditing.

| Feature                  | Change Data Capture | Change Tracking | Temporal Tables |
|--------------------------|---------------------|-----------------|-----------------|
| Track DML Changes       | Yes                | Yes            | Yes            |
| Track Historical Data   | Yes                | No             | Yes            |
| Whether Column Changed  | Yes                | Yes            | No             |
| DML Type                | Yes                | Yes            | No             |
| Azure SQL DB Support    | No                 | Yes            | Yes            |
| SQL Versions Supported  | Various            | Various        | Various        |

Real-world example: Use temporal tables in an inventory system to track price changes over time, allowing auditors to reconstruct historical stock values for tax reporting.

### Implement Dynamic Data Masking

Mask sensitive data for non-privileged users.

- **Rules**: Default (full mask), Email (aXX@XXXX.com), Random (number range), Custom String (prefix/padding/suffix), Credit Card (XXXX-XXXX-XXXX-1234, portal only).

Real-world example: In a call center app, mask credit card numbers for agents, showing only last four digits, preventing exposure while allowing transaction verification.

### Implement Ledger in Azure SQL

Provides tamper-evident, immutable data history using blockchain-like tech.

Real-world example: Implement ledger for supply chain tracking in manufacturing, ensuring shipment records can't be altered, proving authenticity to regulators.

### Implement Row-Level Security

Filter rows based on user attributes.

Real-world example: In a multi-department DB, apply row-level security so HR sees only employee records for their team, enforcing data isolation without multiple DBs.
