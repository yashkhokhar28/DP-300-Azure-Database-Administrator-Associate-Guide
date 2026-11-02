# Module 2: Implement a Secure Environment

**Exam weight:** 20–25% · **Reading time:** ~85 min · **Files:** 8

## What you will learn
You will learn how people and apps prove who they are, and how to give them only the rights they need. You will learn to encrypt data on disk, in chosen columns, and on the network. You will learn to control which networks can reach a database. Finally, you will learn the compliance tools: labels, audits, masking, row filters, ledger, threat alerts, and change tracking.

Picture a bank. The front door checks your ID (authentication). Your key card opens only some rooms (permissions). The vault is locked (TDE), and some safe boxes can be opened only by their owners, not by staff (Always Encrypted). Money moves in armoured vans (TLS) along private roads (Private Link). Cameras record everything (auditing), tellers see only part of a card number (masking), each branch sees only its own customers (row-level security), and a sealed logbook shows if a page was torn out (ledger).

## Lessons in this module
| # | Lesson | What it covers | Time |
|---|--------|----------------|------|
| 1 | [Authentication](<01 - Authentication.md>) | SQL vs Entra ID login, Entra admin, Entra-only auth, managed identities, Azure RBAC | ~10 min |
| 2 | [Users, roles and permissions](<02 - Users, roles and permissions.md>) | Logins, users, roles, GRANT/DENY/REVOKE, ownership chains, least privilege | ~13 min |
| 3 | [Troubleshoot login and permission problems](<03 - Troubleshoot login and permission problems.md>) | Transient faults, retry logic, fixing "login failed" | ~6 min |
| 4 | [Encryption](<04 - Encryption.md>) | TDE, Key Vault keys, cell encryption, Always Encrypted, VBS and SGX enclaves, TLS | ~20 min |
| 5 | [Network security](<05 - Network security.md>) | Firewall rules, service endpoints, Private Link, connection policy, SQL injection | ~8 min |
| 6 | [Compliance controls](<06 - Compliance controls.md>) | Classification, Purview, auditing, masking, row-level security, ledger, Defender for SQL | ~17 min |
| 7 | [Change tracking and change data capture](<07 - Change tracking and change data capture.md>) | CT vs CDC, setup, querying, platform limits | ~10 min |

## Words you will meet
| Word | Simple meaning |
|------|----------------|
| Authentication | Proving who you are |
| Authorization | What you are allowed to do |
| Microsoft Entra ID | Microsoft's cloud identity service (old name: Azure AD) |
| Managed identity | An Azure-managed identity for a resource, with no password to store |
| Login / User | Access to the server / access to one database |
| Least privilege | Give only the rights needed for the job |
| TDE | Encrypts the whole database and backups on disk |
| Always Encrypted | Encrypts chosen columns in the app, so the server never sees plain values |
| Secure enclave | A locked area of server memory where encrypted data can be safely worked on |
| TLS | Encryption for data moving over the network |
| Private Link | Gives the database a private IP inside your virtual network |
| Service endpoint | Lets a chosen subnet reach the database over Azure's backbone |
| Dynamic data masking | Hides part of a value in query results |
| Row-level security | Each user sees only the rows they are allowed to see |
| Ledger | Makes tampering with data detectable |

## Exam skills covered
- [ ] Configure Microsoft Entra ID authentication for Azure SQL Database, Azure SQL Managed Instance, and SQL Server → 1
- [ ] Configure authentication for SQL on Azure VMs and Azure SQL Managed Instance → 1
- [ ] Configure security principals → 2
- [ ] Create users from Microsoft Entra identities → 2
- [ ] Configure database and object-level permissions using graphical tools → 2
- [ ] Apply the principle of least privilege for all securables → 2
- [ ] Troubleshoot authentication and authorization issues → 3
- [ ] Manage authentication and authorization by using T-SQL → 2, 3
- [ ] Implement transparent data encryption (TDE) → 4
- [ ] Implement object-level encryption → 4
- [ ] Configure server- and database-level firewall rules → 5
- [ ] Implement Always Encrypted → 4
- [ ] Implement Always Encrypted with secure enclaves → 4
- [ ] Configure private links and service endpoints → 5
- [ ] Apply a data classification strategy → 6
- [ ] Configure server and database audits → 6
- [ ] Implement change data tracking → 7
- [ ] Implement dynamic data masking → 6
- [ ] Implement ledger in Azure SQL → 6
- [ ] Implement row-level security → 6
