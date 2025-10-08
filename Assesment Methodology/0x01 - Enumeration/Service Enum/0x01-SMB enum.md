# SMB Enumeration (SMB/NetBIOS)

A practical, step-by-step reference for enumerating Windows file sharing (SMB) services during authorized assessments.

- Protocol: Server Message Block (SMB)
- Common ports: 445/TCP (SMB over TCP), 139/TCP (NetBIOS Session Service)
- Samba: Linux/Unix implementation of SMB

> Legal/Ethics: Only test targets you own or are explicitly authorized to assess.

---

## Quick start: What to try first

```bash
# Set a convenience variable for the target IP
export TARGET=10.10.10.10
```

1) Check open ports and versions
```bash
nmap -p139,445 -sV -Pn $TARGET
nmap -p445 --script smb-os-discovery,smb2-security-mode,smb2-time $TARGET
```

2) Try anonymous/null session
```bash
# List shares anonymously (null session)
smbclient -L //$TARGET/ -N
# Alternative null auth form
smbclient -L //$TARGET/ -U ''%''
```

3) Enumerate more details (no creds/with creds)
```bash
# NSE scripts (anonymous where possible)
nmap -p445 --script smb-enum-shares,smb-enum-users,smb-security-mode $TARGET

# smbmap for share listing
smbmap -H $TARGET                 # anonymous
smbmap -H $TARGET -u anonymous -p ""

# enum4linux (classic) or enum4linux-ng
enum4linux -a $TARGET             # if installed
enum4linux-ng -A $TARGET          # newer fork, if installed
```

4) If you have credentials
```bash
# List shares with creds
smbclient -L //$TARGET/ -U USER
# Quick non-interactive (avoid prompt)
smbclient -L //$TARGET/ -U USER%PASS
# Access a share
smbclient //${TARGET}/SHARE -U USER%PASS
```

---

## Discovery and scanning

### Nmap: ports, versions, and scripts (NSE)

```bash
# Fast port check and service versions
nmap -p139,445 -sV -Pn $TARGET

# Operating system and SMB2/3 security mode info
nmap -p445 --script smb-os-discovery,smb2-security-mode,smb2-time $TARGET

# Shares and users (anonymous where possible)
nmap -p445 --script smb-enum-shares,smb-enum-users $TARGET

# Protocol capabilities and dialects
nmap -p445 --script smb-protocols,smb2-capabilities $TARGET

# (Optional) Vulnerability checks — only if authorized
nmap -p445 --script smb-vuln-ms17-010 $TARGET
```

Notes
- Use `-Pn` if ICMP is blocked.
- Some scripts require credentials; provide with `--script-args smbuser=USER,smbpass=PASS`.

---

## Anonymous and authenticated enumeration

### smbclient
`smbclient` is the Swiss-army knife for SMB share enumeration and file access.

```bash
# List shares anonymously (null session)
smbclient -L //$TARGET/ -N

# List shares with a username (will prompt for password)
smbclient -L //$TARGET/ -U USER

# List shares with explicit user:pass (no prompt)
smbclient -L //$TARGET/ -U USER%PASS

# Connect to a share (interactive shell)
smbclient //${TARGET}/SHARE -U USER%PASS
```

Inside smbclient (common commands)
- `help` — list commands
- `ls` / `cd` / `pwd` — navigate
- `get file` / `put file` — download/upload
- `mget *` — multi-download (use `recurse ON`, `prompt OFF` for bulk)
- `lcd <path>` — set local download directory
- `exit` — quit

Tip: If SMBv1 is disabled or the server enforces newer dialects, force a protocol:
```bash
# Try forcing SMB3 or SMB2
auth SMB3: smbclient -m SMB3 -L //$TARGET/ -U USER%PASS
auth SMB2: smbclient -m SMB2 -L //$TARGET/ -U USER%PASS
```

### smbmap
`smbmap` quickly enumerates share permissions and accessibility.

```bash
# Anonymous checks
smbmap -H $TARGET
smbmap -H $TARGET -u anonymous -p ""

# With credentials
smbmap -H $TARGET -u USER -p PASS

# Recursively list a specific share
smbmap -H $TARGET -u USER -p PASS -r SHARE

# Save file lists to disk
smbmap -H $TARGET -u USER -p PASS -r SHARE --download '*'   # careful: may be large
```

### rpcclient
`rpcclient` enumerates users, groups, and policies via MS-RPC.

```bash
# Null session
rpcclient -U "" -N $TARGET
# Authenticated
rpcclient -U USER%PASS $TARGET
```
Once connected, useful commands:
- `enumdomusers` — list domain users
- `queryuser <RID>` — details for a specific user
- `enumdomgroups` — list groups
- `enumalsgroups domain` — list alias groups
- `lookupnames <name>` — resolve names to RIDs
- `lsaquery` — domain info

### enum4linux / enum4linux-ng
Automates many of the above checks.

```bash
enum4linux -a $TARGET
# or
enum4linux-ng -A $TARGET
```

---

## Metasploit approach (optional)

Start Metasploit and create a workspace (optional but recommended):
```bash
msfconsole
# inside msfconsole
workspace -a smb_enum
```

Common SMB auxiliary modules:

1) Version detection
```text
use auxiliary/scanner/smb/smb_version
set RHOSTS $TARGET
set RPORT 445
run
```

2) Enumerate users
```text
use auxiliary/scanner/smb/smb_enumusers
set RHOSTS $TARGET
set RPORT 445
# Optional creds if anonymous fails
# set SMBUser USER
# set SMBPass PASS
run
```

3) Enumerate shares
```text
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS $TARGET
set RPORT 445
set ShowFiles true   # also list files on accessible shares
# Optional creds
# set SMBUser USER
# set SMBPass PASS
run
```

4) Password/spray login
```text
use auxiliary/scanner/smb/smb_login
set RHOSTS $TARGET
set RPORT 445
set USER_FILE users.txt         # or set USERNAME USER
set PASS_FILE passwords.txt     # or set PASSWORD PASS
set STOP_ON_SUCCESS true
set THREADS 10
run
```

> Tip: Use results from `smb_version` and `smb_enumusers` to guide next steps.

---

## Password spraying and brute-force (use with extreme caution)

Prefer targeted password spraying over brute-force, and always respect rate limits and rules of engagement.

### Metasploit (shown above)
Use `auxiliary/scanner/smb/smb_login` with small, vetted lists.

### CrackMapExec (if installed)
```bash
# Basic host info
crackmapexec smb $TARGET

# Enumerate shares anonymously
crackmapexec smb $TARGET -u '' -p '' --shares

# With credentials
crackmapexec smb $TARGET -u USER -p PASS --shares
crackmapexec smb $TARGET -u USER -p PASS --users
```

### Hydra (falls back to SMB support; results vary by version)
```bash
# Hydra's smb module may target older SMB; prefer Metasploit/CME for SMBv2/3
hydra -L users.txt -P passwords.txt smb://$TARGET
```

---

## After credentials: access and data collection

### Interactive access with smbclient
```bash
# Connect to share
smbclient //${TARGET}/SHARE -U USER%PASS

# In the smbclient shell
lcd ~/loot/smb-$TARGET
recurse ON
prompt OFF
mget *
exit
```

### Mount a share (Linux)
```bash
sudo mkdir -p /mnt/smbshare
sudo mount -t cifs //${TARGET}/SHARE /mnt/smbshare \
  -o username=USER,password=PASS,vers=3.0,uid=$(id -u),gid=$(id -g)

# After use
sudo umount /mnt/smbshare
```

If mount fails, try a different `vers=` (e.g., `3.1.1`, `2.1`). Avoid SMB1 unless absolutely necessary.

---

## Troubleshooting and edge cases

- SMBv1 disabled: Force a newer dialect with `smbclient -m SMB3 ...` or `-m SMB2`.
- Domain vs local accounts: In some tools, specify domain as `DOMAIN/USER` or `USER@DOMAIN`.
- Firewalls: ICMP may be blocked — add `-Pn` to Nmap; ports may be filtered.
- Account lockout: Mind password policy; avoid lockouts with aggressive spraying.
- Time skew: Kerberos/AD environments are sensitive to clock drift; ensure your system time is correct.

---

## Quick cheat sheet

```bash
# Ports and version
nmap -p139,445 -sV -Pn $TARGET
nmap -p445 --script smb-os-discovery,smb2-security-mode $TARGET

# Anonymous shares
smbclient -L //$TARGET/ -N
smbmap -H $TARGET -u anonymous -p ""

# Users and shares via NSE
nmap -p445 --script smb-enum-users,smb-enum-shares $TARGET

# rpcclient
rpcclient -U "" -N $TARGET    # then: enumdomusers, enumdomgroups

# With creds
smbclient -L //$TARGET/ -U USER%PASS
smbclient //${TARGET}/SHARE -U USER%PASS
smbmap -H $TARGET -u USER -p PASS -r SHARE

# Metasploit
use auxiliary/scanner/smb/smb_version
use auxiliary/scanner/smb/smb_enumusers
use auxiliary/scanner/smb/smb_enumshares
use auxiliary/scanner/smb/smb_login
```

---

## Notes
- Documentation assumes Linux attacker machine and typical tooling.
- Replace `USER`, `PASS`, `DOMAIN`, `SHARE`, and `$TARGET` with actual values.
- Keep detailed notes of shares, permissions, and interesting files for reporting.

