## Import Nmap results into Metasploit (msfconsole)

This guide shows how to export Nmap scans to XML, import them into Metasploit Framework (msfconsole), verify the import, and perform scans from inside msfconsole. It includes example commands, expected outputs, and troubleshooting tips.

### 1) Generate Nmap XML output
Run Nmap externally and save the results as XML. Example:

```bash
# save XML and also normal text & grepable with -oA
nmap -Pn -sV -O -oA scans/target1 192.168.1.10
# This produces: scans/target1.nmap, scans/target1.xml, scans/target1.gnmap
```

Notes:
- Use `-Pn` if ICMP is filtered and you want to skip host discovery.
- `-sV` for version detection, `-O` for OS detection (requires root).

### 2) Ensure Metasploit DB (PostgreSQL) is running
Metasploit uses PostgreSQL to store imported scan results. Start the DB service and then launch msfconsole:

```bash
# start postgres (example for Debian/Ubuntu)
sudo service postgresql start

# start msfconsole
msfconsole
```

Inside msfconsole you can check DB status:

```
msf6 > db_status
[*] postgresql connected to msf
```

If `db_status` says `postgresql not connected`, ensure the service is running and Metasploit is configured to use it (check `~/.msf4/database.yml`).

### 3) Workspaces (optional but recommended)
Workspaces let you separate projects. List, create, and switch workspaces inside msfconsole:

```
msf6 > workspace
Workspace: my_default

msf6 > workspace -a target1_proj
[+] Added workspace: target1_proj

msf6 > workspace
Workspace: target1_proj
```

Switch back to a workspace with `workspace <name>`.

### 4) Import the Nmap XML
From msfconsole, import the XML file produced by Nmap:

```
msf6 > db_import /full/path/to/scans/target1.xml
[+] Importing: /full/path/to/scans/target1.xml
[+] Hosts: 1
[+] Services: 7
[+] Vulns: 0
```

If the file is large the import will take longer. Use the absolute path to avoid confusion.

### 5) Verify imported data
Common commands to inspect imported information:

```
msf6 > hosts
Hosts
=====

address         mac            name         os_name  purpose
-------         ---            ----         -------  -------
192.168.1.10                  target1.local Linux    -

msf6 > services
Services
========

host           port proto name     state  info
----           ---- ----- ----     -----  ----
192.168.1.10   80   tcp   http     open   Apache httpd 2.4.29
192.168.1.10   22   tcp   ssh      open   OpenSSH 7.6p1

msf6 > vulns
Vulnerabilities
===============

host           port  name  info
----           ----  ----  ----
```

Other useful database commands:
- `notes` — view collected notes
- `loot` — view uploaded/downloaded files
- `creds` — list discovered credentials

### 6) Perform Nmap from inside msfconsole (alternative)
You can run Nmap directly from msfconsole with `db_nmap` (this both runs nmap and imports results):

```
msf6 > db_nmap -Pn -sV -p1-1000 192.168.1.0/24
[+] Nmap run complete, importing results
[+] Hosts: 42
```

`db_nmap` is convenient but requires Nmap to be installed and in PATH on the system running msfconsole.

### 7) Common troubleshooting
- db_import fails/empty: ensure XML is valid (open in a text editor), use `nmap -oX -` to print XML to stdout and verify.
- `db_status` shows not connected: start PostgreSQL and confirm Metasploit `database.yml` points to the right socket/port.
- Permission denied importing file: use a path msf can access or move the file to a shared location (e.g., `/tmp/scans/`).
- Missing services after import: re-run `nmap` with `-sV` and `-oX` to ensure version info is captured.

### 8) Practical workflow example

1. From your attack machine run:

```bash
sudo nmap -Pn -sV -O -oA /tmp/scans/target1 192.168.1.10
```

2. Start PostgreSQL and msfconsole:

```bash
sudo service postgresql start
msfconsole
```

3. Create workspace and import:

```
msf6 > workspace -a target1
msf6 > db_import /tmp/scans/target1.xml
msf6 > hosts; services; vulns
```

4. Continue enumeration based on imported services (set up auxiliary modules, run credential checks, etc.).

### 9) Security & ethics notes
- Only import scans and run Metasploit actions against targets you are authorized to test.
- Importing large, noisy scans into a shared Metasploit DB can produce a lot of data — clean up unused workspaces when finished.

---
Revision: Detailed steps to export Nmap XML and import into Metasploit Framework (msfconsole).
