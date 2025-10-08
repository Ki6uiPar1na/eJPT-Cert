# SMB Enumeration (SMB/NetBIOS) — quick reference and Metasploit details

Short notes:
- Protocol: SMB (Server Message Block)
- Common ports: 445/TCP (SMB over TCP), 139/TCP (NetBIOS session)
- Samba: Linux implementation of the SMB/CIFS protocol

Always have explicit authorization before testing.

---

## Quick useful commands (outside Metasploit)

```bash
export TARGET=10.10.10.10
# Port & basic version scan
nmap -p139,445 -sV -Pn $TARGET

# NSE scripts for SMB discovery
nmap -p445 --script smb-os-discovery,smb-enum-shares,smb-enum-users,smb2-security-mode $TARGET

# Try anonymous share listing
smbclient -L //$TARGET/ -N

# Quick share map (smbmap)
smbmap -H $TARGET

# Enumerate lots of SMB info (enum4linux)
enum4linux -a $TARGET
```

---

## NetBIOS name lookup: nmblookup

Use `nmblookup` (part of the Samba tools) to query NetBIOS names and discover the host's NetBIOS name, workgroup/domain, and registered services. This is a lightweight, fast pre-check before deeper SMB/SMB2/SMB3 enumeration.

Common commands

```bash
# Query NetBIOS name table by host IP (recommended)
nmblookup -A $TARGET   # note: lowercase 'nmblookup' is common; some systems just use 'nmblookup'
# Example: nmblookup -A 10.10.10.10

# Query a NetBIOS name via broadcast/WINS (resolve a name to IP)
nmblookup NAME          # e.g. nmblookup WORKGROUP or nmblookup TARGET-BOX

# Windows equivalent (if on Windows):
# nbtscan or nbtstat -A <ip>
```

Example output (from `nmblookup -A 10.10.10.10`)

```
Looking up status of 10.10.10.10
        WORKGROUP        <00> -         B <ACTIVE>
        TARGET-BOX       <00> -         B <ACTIVE>
        TARGET-BOX       <20> -         B <ACTIVE>

        MAC Address = 00:0c:29:ab:cd:ef
```

Interpreting the output
- The left column contains NetBIOS names registered by the host (workgroup/computer name).
- The numeric suffix (e.g., `<00>`, `<20>`) is the NetBIOS service type:
  - `<00>` — workstation/service name (common host name entry)
  - `<20>` — file server service (SMB server)
  - Other suffixes exist (e.g., browser/master roles) — consult Samba docs for a full list.
- `MAC Address` is often shown with `-A` and can help identify VM vs physical hosts.

Why this is useful
- Confirms the host advertises SMB/File Server service (`<20>`).
- Reveals NetBIOS name and workgroup/domain used by the target — helpful for credential targeting and AD-aware enumeration.
- Fast and quiet compared to a full Nmap scan; good initial reconnaissance step.

Notes
- `nmblookup` requires network reachability to the NetBIOS service (port 137/UDP for name queries, and `-A` contacts the host directly).
- Modern environments may deprecate NetBIOS (WINS) in favor of DNS + SMB over TCP (445); if NetBIOS is disabled, `nmblookup` may return little or nothing.

---

## rpcclient (MS-RPC enumeration)

`rpcclient` is a powerful Samba tool that connects to the Microsoft RPC service exposed over SMB and provides many enumeration commands. It often succeeds where anonymous SMB listing fails and can enumerate domain users, groups, shares, and a range of other useful information.

Basic usage

```bash
# Null-session (anonymous) connection
rpcclient -U "" -N $TARGET

# Authenticated connection (username and password)
rpcclient -U 'USER%PASS' $TARGET

# Authenticated with domain (DOMAIN\USER)
rpcclient -U 'DOMAIN\\USER%PASS' $TARGET
```

Start an interactive session and run commands

```bash
# Connect anonymously and drop into an rpcclient prompt
rpcclient -U "" -N 10.10.10.10

# At the rpcclient prompt, run commands like:
srvinfo                 # show server info and OS hints
enumdomusers            # enumerate domain users
enumdomgroups           # enumerate domain groups
lookupnames <name>      # resolve NetBIOS name(s) to RIDs/IPs
lookuprid <RID>         # map RID back to a name
queryuser <RID|name>    # user details (if available)
netshareenum            # list shares via RPC
netsharegetinfo SHARE   # detailed info about a share
lsaquery                # query LSA/Domain info

# Example sequence
rpcclient $> srvinfo
rpcclient $> enumdomusers
rpcclient $> lookupnames administrator
rpcclient $> netshareenum
```

Example session (trimmed)

```
$ rpcclient -U "" -N 10.10.10.10
rpcclient $> srvinfo
Server OS: Windows 6.1 (Build 7601: Service Pack 1)
Server Name: TARGET-BOX
Domain Name: WORKGROUP

rpcclient $> enumdomusers
RID: 0x1f4 Username: Administrator
RID: 0x1f5 Username: Guest
RID: 0x1f6 Username: john

rpcclient $> netshareenum
Share: ADMIN$
Share: C$
Share: public
```

When to use `rpcclient`
- Use it when `smbclient` or NSE scripts don't show user or domain information.
- It is particularly useful against domain-joined hosts and for enumerating Active Directory-related info when SMB is available.

Notes and caveats
- Some commands require elevated privileges or a domain context to return full results.
- Anonymous/null sessions may be disabled; use valid credentials when needed.
- `rpcclient` output is textual and intended to be inspected; redirect output to files for later parsing.

---

## Metasploit: workflow, commands, and explanations

This section expands and corrects the msfconsole workflow from basic discovery through enumeration and credential testing. Replace `10.10.10.10` with your target IP and adjust `USER`, `PASS` or file paths as needed.

### 1) Start Metasploit and database

Open msfconsole and ensure the database is initialized and connected (important for workspaces and storing loot):

```text
#start the database
sudo service postgresql start  
```

```text
# start msfconsole from shell
msfconsole

# inside msfconsole (ensure database is online)
msf6 > db_status
# if not connected:
msf6 > db_connect
# (on modern installs this is often automatic)
```

Create or switch to a workspace to keep data isolated:

```text
# create a new workspace
msf6 > workspace -a smb_enum
# switch to the workspace
msf6 > workspace smb_enum
msf6 > workspace
```

You can use `workspace` to separate engagements.

### 2) SMB version detection

Module: auxiliary/scanner/smb/smb_version
Purpose: identify SMB dialect, NetBIOS name, OS details — useful to choose later modules.

```
msf6 > search type:auxiliary smb_version
msf6 > use auxiliary/scanner/smb/smb_version
msf6 auxiliary(smb_version) > show options

# Required options
# RHOSTS: target(s)
# RPORT: 445 (default)

msf6 auxiliary(smb_version) > set RHOSTS 10.10.10.10
msf6 auxiliary(smb_version) > set THREADS 10     # optional: parallelize
msf6 auxiliary(smb_version) > run
```

Example (sample) output you might see (trimmed):

```
[*] 10.10.10.10:445    - Host is likely running Windows 7 or Server 2008 R2
[*] 10.10.10.10:445    - Protocol: SMBv2, dialect: 3.0.0
[*] 10.10.10.10:445    - NetBIOS Computer Name: TARGET-BOX
```

Use this info to decide whether SMBv1-only tools are useful or whether you must target SMBv2/3.

### 3) Enumerate users

Module: auxiliary/scanner/smb/smb_enumusers
Purpose: attempt to enumerate user accounts via SMB methods (null session, RPC, etc.).

Notes: Some targets will not reveal users anonymously. If the target requires authentication, provide `SMBUser` and `SMBPass`.

```
msf6 > search type:auxiliary smb_enumusers
msf6 > use auxiliary/scanner/smb/smb_enumusers
msf6 auxiliary(smb_enumusers) > show options

# Typical options
# RHOSTS, RPORT
# THREADS
# SMBUser, SMBPass (optional)

msf6 auxiliary(smb_enumusers) > set RHOSTS 10.10.10.10
msf6 auxiliary(smb_enumusers) > run
```

Sample output (what to look for):

```
[*] Enumerating users for 10.10.10.10
[*] Found: Administrator
[*] Found: Guest
[*] Found: john
[*] Found: svc_backup
```

If `smb_enumusers` returns accounts, export/save the list for later password spraying or targeted brute force.

### 4) Enumerate shares and optionally list files

Module: auxiliary/scanner/smb/smb_enumshares
Purpose: list SMB shares and optionally enumerate files within accessible shares (ShowFiles option).

```
msf6 > use auxiliary/scanner/smb/smb_enumshares
msf6 auxiliary(smb_enumshares) > show options
msf6 auxiliary(smb_enumshares) > set RHOSTS 10.10.10.10
msf6 auxiliary(smb_enumshares) > set ShowFiles true  # attempts to list files in readable shares
msf6 auxiliary(smb_enumshares) > run
```

Sample output highlights:

```
[*] 10.10.10.10:445 - Enumerating shares
[+] Share: ADMIN$  - Admin share (no listing)
[+] Share: C$      - Admin hidden share
[+] Share: public  - Readable
[*] Listing files in share public:
    - report.pdf
    - password_backup.txt
```

Important: Listing files may require credentials; anonymous enumeration might only show share names.

### 5) Try logins / password testing

Module: auxiliary/scanner/smb/smb_login
Purpose: attempt authentication using single credentials or lists (user/pass files). Use `STOP_ON_SUCCESS true` to halt when credentials are found.

```
msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(smb_login) > show options

# Important options
# RHOSTS, RPORT
# USERNAME or USER_FILE
# PASSWORD or PASS_FILE
# STOP_ON_SUCCESS (true/false)
# USER_AS_PASS (try username as password)
# THREADS

msf6 auxiliary(smb_login) > set RHOSTS 10.10.10.10
# Single credential test
msf6 auxiliary(smb_login) > set SMBUser administrator
msf6 auxiliary(smb_login) > set SMBPass 'P@ssw0rd'
msf6 auxiliary(smb_login) > run

# Brute/spray from lists (recommended: small list, observe lockout policy)
msf6 auxiliary(smb_login) > set USER_FILE /path/to/users.txt
msf6 auxiliary(smb_login) > set PASS_FILE /path/to/passwords.txt
msf6 auxiliary(smb_login) > set STOP_ON_SUCCESS true
msf6 auxiliary(smb_login) > set THREADS 10
msf6 auxiliary(smb_login) > run
```

Sample success output:

```
[*] 10.10.10.10:445 - SUCCESSFUL LOGIN: 10.10.10.10:445 - administrator:P@ssw0rd
[+] Credentials saved to the database
```

After a successful authentication, export or note credentials immediately for follow-up actions.

### 6) Using the credentials with Metasploit or external tools

Once you have a valid username/password pair, you can use Metasploit post-auth modules or external tools like `smbclient`, `smbmap`, or `crackmapexec` to interact with shares.

Example: list shares via `smbclient` using discovered creds

```bash
smbclient -L //10.10.10.10/ -U administrator%P@ssw0rd
smbclient //10.10.10.10/public -U administrator%P@ssw0rd
```

Example: use the creds in Metasploit to run an authenticated scanner or a post module

```
# set creds as module options
set SMBUser administrator
set SMBPass P@ssw0rd
# then run other smb_* auxiliary or post modules that accept SMBUser/SMBPass
```

### 7) Additional Metasploit tips & good practices

- Use `set RHOSTS` for ranges and `RHOST` for single host depending on module; `show options` lists required variables.
- `set THREADS` can speed up scanning but be careful to avoid noisy DoS-like behavior.
- Use `workspace` to separate target data and `db_export`/`db_import` to move data.
- Save credentials into Metasploit's DB with `creds` module integration (often automatic on success) for later use by other modules.
- Respect account lockout policies — prefer password spraying with long delays rather than brute-force.

---

## Example end-to-end (short flow)

1. Start msfconsole and workspace

```
msf6 > workspace -a smb_test
msf6 > workspace smb_test
```

2. Detect SMB version

```
msf6 > use auxiliary/scanner/smb/smb_version
msf6 auxiliary(smb_version) > set RHOSTS 10.10.10.10
msf6 auxiliary(smb_version) > run
```

3. Enumerate users

```
msf6 > use auxiliary/scanner/smb/smb_enumusers
msf6 auxiliary(smb_enumusers) > set RHOSTS 10.10.10.10
msf6 auxiliary(smb_enumusers) > run
```

4. Enumerate shares

```
msf6 > use auxiliary/scanner/smb/smb_enumshares
msf6 auxiliary(smb_enumshares) > set RHOSTS 10.10.10.10
msf6 auxiliary(smb_enumshares) > set ShowFiles true
msf6 auxiliary(smb_enumshares) > run
```

5. Attempt login (from a small safe list)

```
msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(smb_login) > set RHOSTS 10.10.10.10
msf6 auxiliary(smb_login) > set USER_FILE /root/users.txt
msf6 auxiliary(smb_login) > set PASS_FILE /root/passwords.txt
msf6 auxiliary(smb_login) > set STOP_ON_SUCCESS true
msf6 auxiliary(smb_login) > set THREADS 5
msf6 auxiliary(smb_login) > run
```

---

## Post-auth interaction (examples outside Metasploit)

```bash
# Using smbclient to list and access shares
smbclient -L //10.10.10.10/ -U administrator%P@ssw0rd
smbclient //10.10.10.10/public -U administrator%P@ssw0rd

# Mount a share on Linux (CIFS)
sudo mount -t cifs //10.10.10.10/public /mnt/smb -o username=administrator,password=P@ssw0rd,vers=3.0
```

---

## Safety and troubleshooting

- If a module complains about missing options, run `show options` and set variables as required.
- Some modules expect `RHOSTS` (list/range) vs `RHOST` (single IP) — read `show options` carefully.
- If enumeration fails without creds, try authenticated scans (if you have legitimate creds) or use other tools (`rpcclient`, `enum4linux`, `smbmap`).
- Be cautious with `set THREADS` and large password lists — watch account lockout policies and target stability.

---

Keep this file as your quick SMB enumeration reference and adjust commands to match your local tool paths and policy constraints.

