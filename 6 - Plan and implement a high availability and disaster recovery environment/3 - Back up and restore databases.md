# 🧠 Backup & Restore in Azure SQL: Real Talk

## 🚨 Why You Can’t Skip Backups

Even if you’re not rockin’ high-end DR/HA setups like:

- ✅ Always On AGs
- ✅ Geo-replication

You still **must** have a **backup + restore plan**. Why?

### 💥 Real-world failure scenarios:

- **Human oops:** Dev runs a `DELETE FROM Users` in prod. Poof 💨
- **Ransomware:** Your Azure VM is locked down until you pay ₹20L in crypto.
- **Cloud region outage:** Lightning fries the data center (literally).
- **App bug:** App migration corrupts 20% of your data.

👉 In all cases, **backups are your parachute**. Without them? You’re free-falling into chaos.

---

## 🧱 What a Solid Backup Strategy Looks Like (Real Examples)

| Part              | What It Means                                          | Real-World Scenario                                         |
| ----------------- | ------------------------------------------------------ | ----------------------------------------------------------- |
| **Backup types**  | On-prem, offsite, or cloud backups                     | Backup to Azure Blob to survive region failure              |
| **Automation**    | No cron-jobs in Notepad please — use proper scheduling | SQL IaaS Agent Extension for automated nightly backups      |
| **Recovery plan** | Know how to restore fast, not just take backups        | Dev drops prod DB at 2AM → You restore before the CEO wakes |

---

## 🎯 Real Goal: Business Continuity

Backups aren’t just data insurance — they’re **revenue protection**. Think downtime = lost sales, angry customers, compliance violations.

> “No one cares about your uptime… until you go down.”

---

## 🧠 Azure VM SQL: You're on Your Own

When you run SQL on an **Azure VM**, backups are **not automatic** like in PaaS. You’re the DBA now, buddy.

You must back up:

- **System DBs:** `master`, `msdb` (not usually restored elsewhere).
- **User DBs:** Actual app data — this is your golden goose 🥚.

---

## 🧩 SQL Server Backup Types (With Examples)

| Backup Type         | Description                            | Example Use Case                         |
| ------------------- | -------------------------------------- | ---------------------------------------- |
| **Full**            | Entire DB snapshot                     | Nightly at 2 AM                          |
| **Differential**    | Only what changed since last full      | Every 6 hours                            |
| **Transaction Log** | Every committed change, log truncation | Every 15 minutes for near-zero data loss |

🛠 Want to restore to 10:35 AM? You need:

1. Full
2. Differential (if any)
3. All logs up to 10:35

---

## 🔄 SQL Server Recovery Models (With TL;DR)

| Model           | Point-in-time restore? | Used When…                                 |
| --------------- | ---------------------- | ------------------------------------------ |
| **FULL**        | ✅ Yes                 | You care about not losing a single row     |
| **SIMPLE**      | ❌ No                  | Logging isn’t important (dev/test/staging) |
| **BULK_LOGGED** | ✅ Sorta               | Bulk inserts, speed prioritized            |

---

## ☁️ Backup Options in Azure VM (SQL IaaS)

### 1. **Azure VM Snapshot (Full VM Backup)**

- Great for: Entire VM recovery (ransomware, disaster)
- Not great for: Restoring one DB to 10:32 AM

> 🧠 Think of this like a full phone backup. You can’t restore just your WhatsApp chat.

⚙️ Got snapshot issues? Add this to the registry:

```reg
[HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\BCDRAGENT]
"USEVSSCOPYBACKUP"="TRUE"
```

---

### 2. **SQL Native Backups to Disk / Azure Files**

- Write backups to local disk or mounted share.
- 🔁 Copy to offsite/cloud (don’t trust one location).

> Example: `.bak` files dumped daily to Azure Files + mirrored to Blob.

---

### 3. **Backup to Azure Blob (URL Backup)**

- Native SQL Server support
- Cheap, secure, cloud-native

```sql
BACKUP DATABASE contoso
TO URL = 'https://myacc.blob.core.windows.net/db/contoso.bak'
```

> ✅ Easy restore using `RESTORE DATABASE FROM URL`

---

### 4. **Automated Backups via SQL IaaS Agent Extension**

- Full, differential, log — all managed for you
- Stored in Azure Blob
- Retention policies built-in

⚠️ **DO NOT** mix this with manual backups, or you’ll break the log chain and cry later during a restore.

---

## 🔁 Restoring: What It Looks Like

| Method             | Manual Restore Needed? | Ideal For                       |
| ------------------ | ---------------------- | ------------------------------- |
| Azure VM Snapshot  | ✅ Yes                 | Full VM recovery                |
| Disk / Azure Files | ✅ Yes                 | Precise DB recovery             |
| Azure Blob         | ✅ Yes                 | Geo-resilient, granular restore |

> You always run SQL scripts or SSMS — no "Restore Button" magic.

---

## 💡 PaaS SQL (Azure SQL DB / Managed Instance)

### Automatic Backups Include:

- Full → Weekly
- Differential → Every 12h
- Log → Every 5–10 min

Stored in **geo-redundant storage (RA-GRS)**. Region burns down? You're still good 🔥🧯

---

### PaaS Restore Scenarios:

| Action                 | Available? | Details                               |
| ---------------------- | ---------- | ------------------------------------- |
| Point-in-time restore  | ✅ Yes     | Any second within 7–35 days           |
| Restore deleted DB     | ✅ Yes     | Must give a new name                  |
| Restore in place       | ❌ No      | Create new DB only                    |
| Manual `BACKUP TO URL` | ✅ (MI)    | Only in Managed Instance (not SQL DB) |

---

## 🚨 Critical Reminders

- **IaaS = You’re the backup boss.**
- **PaaS = Microsoft handles it, but you must know how to restore.**
- **Don't mix backup methods — keep your log chain holy.**
- **Copy backups offsite** — never trust one region or disk.

---

## 🏁 TL;DR Cheat Sheet

| Scenario                         | Best Backup Option                   |
| -------------------------------- | ------------------------------------ |
| Recover whole VM from ransomware | Azure VM Snapshot                    |
| Restore DB to 10:47 AM           | SQL log backups (Blob or disk)       |
| Hands-off backup automation      | SQL IaaS Extension                   |
| Regulatory compliance (LTR)      | Azure PaaS + Long-Term Retention     |
| PaaS DB rollback to yesterday    | Azure Portal → Restore point-in-time |

---
