# Troubleshoot login and permission problems

> **Exam skills:** Troubleshoot authentication and authorization issues
> **Reading time:** ~6 min

## The big idea
When someone cannot connect or cannot read data, check things in order. Is it a short Azure hiccup? Is the network blocked? Does the login exist? Does the user have rights? It is like a car that will not start: check the fuel, then the battery, then the key.

## Common reasons a connection fails
1. **Reconfiguration events.** Azure moves or updates your database. It restarts for a moment. Usually **under 60 seconds**.
2. **Firewall.** No firewall rule allows the client's IP (lesson 5).
3. **Login problems.** Wrong username or password, the login is **disabled**, or it **does not exist**.
4. **Resource limits.** The database hit its CPU, memory, storage or connection limits.
5. **Timeouts.** Slow network or long-running queries.

## Transient faults
A **transient fault** is a short, temporary error. It fixes itself. It happens during planned maintenance, scaling, failover, or (rarely) unplanned problems. Azure reconnects within about **60 seconds**.

Typical transient error messages:
- "Can't open database requested by login. The login failed."
- "Can't process request. Not enough resources to process request."
- "Too many operations in progress for subscription."

**Example:** A pizza-ordering app shows "login failed" for 20 seconds at 3 AM, then works again. Azure was patching the server. This is a transient fault, not a broken password.

### Retry logic
Apps should **retry** quietly, not show the error to users. Microsoft recommends:
- Wait **5 seconds** before the first retry.
- Use **exponential backoff**: wait longer each time, e.g. 5s → 10s → 20s → 40s, up to **60s**.
- Set a **maximum** number of retries, so it never loops forever.
- If a query fails, **close the connection and open a new one** for the retry. Do not reuse the broken connection.

### What to check

| Problem | Action |
|---------|--------|
| Login failures for many users | Check **Azure Service Health** for outages |
| Resource limit errors | Watch CPU, memory, storage; **scale up** if needed |
| Failures lasting **over 60 seconds** | Contact **Azure Support** |

## Fix "Login failed for user"
Work through these steps.

1. **Is the login disabled?**
   ```sql
   SELECT name, is_disabled FROM sys.sql_logins WHERE name = 'Ravi';
   ALTER LOGIN Ravi ENABLE;
   ```
2. **Does the login exist?** If not, create it.
   ```sql
   CREATE LOGIN Ravi WITH PASSWORD = 'StrongP@ss1';
   ```
3. **Is there a user in the target database?** A login alone is not enough.
   ```sql
   CREATE USER Ravi FOR LOGIN Ravi;
   ```
4. **Does the user have permissions?** Add to a role or grant rights.
   ```sql
   ALTER ROLE db_datareader ADD MEMBER Ravi;
   -- or
   GRANT SELECT ON SCHEMA::Sales TO Ravi;
   ```

**Example:** A new analyst can log in to the server but gets an error opening `SalesDB`. The login exists, but nobody created a user in `SalesDB`. Step 3 fixes it.

Note: Failed **Microsoft Entra** logins do **not** show in the SQL audit log. Check the **Entra sign-in logs** for those.

## Authorization errors (logged in, but "permission denied")
- Check for a **DENY** somewhere (direct or through a role). DENY beats GRANT.
- If a procedure uses **dynamic SQL**, the caller needs rights on the tables (ownership chain is broken).
- Test as that user: `EXECUTE AS USER = 'Ravi'; SELECT ...; REVERT;`.

## Check the connection string
Update the app's **connection string** if you:
- created a new database,
- renamed or moved the server, or
- rotated passwords.

Copy a fresh one from the Azure portal: **SQL database → Connection strings**. Then add the username, password and correct server name. For secure connections, keep `Encrypt=True;TrustServerCertificate=False;`.

## Remember for the exam
- Most short "login failed" or "not enough resources" errors are **transient**; they clear in under **60 seconds**.
- Retry: wait **5s** first, then **exponential backoff**, with a **max** retry count.
- Retry on a **new connection**, not the failed one.
- Over 60 seconds of failures → **Azure Support**. Many users failing → **Azure Service Health**.
- "Login failed" steps: login enabled? → login exists? → **user** in the database? → **permissions**?
- `sys.sql_logins` shows `is_disabled`; fix with `ALTER LOGIN ... ENABLE`.
- Failed Entra logins → **Entra sign-in logs**, not the SQL audit.

## Quick check
1. An app gets "Not enough resources to process request" for 30 seconds during scaling. What should the app do?
   <details><summary>Answer</summary>Retry with exponential backoff on a new connection. It is a transient fault.</details>
2. A login exists and is enabled, but the user gets an error opening the database. What is the most likely missing step?
   <details><summary>Answer</summary>There is no database user mapped to the login. Create one with CREATE USER ... FOR LOGIN.</details>
3. Connection failures have lasted 10 minutes and Azure Service Health shows no outage. What next?
   <details><summary>Answer</summary>Contact Azure Support. Failures over 60 seconds are not normal transient faults.</details>
