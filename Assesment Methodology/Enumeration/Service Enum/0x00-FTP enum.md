what is ftp and port of it ? 
need authentication

Demo: 
# FTP Enumeration and Exploitation — Practical Guide

This document covers FTP (File Transfer Protocol) enumeration techniques commonly used during penetration tests and red-team assessments. It includes discovery with Nmap and NSE scripts, banner grabbing, anonymous checks, brute forcing credentials (Metasploit and Hydra), and notes on common FTP vulnerabilities and mitigations.

## Quick facts

- Service name: FTP
- Default port: 21/tcp (control). Data connections use ephemeral ports; in passive mode the server listens on a port negotiated with the client.
- Purpose: file transfer and remote file management. Many implementations support anonymous login and file upload, which often lead to information disclosure or further compromise.

## Goals of FTP enumeration

- Discover whether FTP is running (open port and service/version)
- Check for anonymous access and writable directories
- Gather banner and version info for vulnerability lookup
- Attempt credential discovery (valid accounts) via credential stuffing or brute force
- Identify service-specific vulnerabilities (e.g., backdoors, misconfigurations)

## 1) Service discovery (Nmap)

Start with a service/version scan and targeted NSE scripts for FTP:

```
# quick service version scan and common FTP NSE scripts
nmap -sS -sV -p 21 --script=ftp-anon,ftp-syst,ftp-brute,ftp-vsftpd-backdoor <target>

# scan a range of ports if you suspect non-standard FTP ports
nmap -sS -sV -p- --open --script=ftp-* <target>
```

- `-sS` performs a TCP SYN scan (fast, stealthy). `-sV` attempts to detect service and version. Adjust according to rules of engagement.
- Useful NSE scripts: `ftp-anon` (anonymous access), `ftp-syst` (SYST response), `ftp-brute` (simple brute force), `ftp-vsftpd-backdoor`, `ftp-proftpd-backdoor` (checks for specific historical backdoors). There are others — check `ls /usr/share/nmap/scripts | grep ftp` on your machine.

## 2) Banner grabbing and manual checks

Grab banner information using netcat or telnet:

```
nc -nv <target> 21
# or
telnet <target> 21
```

Look for service name/version in the banner (e.g., vsftpd 2.3.4). Use that string to search exploit databases (Exploit-DB, Metasploit, CVE) to find relevant vulnerabilities.

## 3) Anonymous login check

Many FTP servers allow anonymous logins (username `anonymous` or `ftp`), sometimes with write permissions. Check using the `ftp` client or `curl`:

```
ftp <target>
# when prompted, try username: anonymous  and password: any email or anonymous

# or using curl to list root directory (passive/active behavior depends on server)
curl ftp://anonymous@<target>/
```

If anonymous is allowed, check whether you can upload files or list sensitive folders.

## 4) Metasploit enumeration workflow

Start `msfconsole` and work with auxiliary modules to enumerate FTP from within Metasploit's database and workflows.

```
sudo service postgresql start   # ensure msf database is up
msfconsole
workspace -a ftp_enum
```

Discovery via Metasploit (quick):

```
msf6 > search type:auxiliary ftp
```

Common auxiliary modules you'll find and use:
- `auxiliary/scanner/ftp/anonymous` — check for anonymous login
- `auxiliary/scanner/ftp/ftp_version` or `auxiliary/scanner/ftp/ftp_syst` — version/banner checks
- `auxiliary/scanner/ftp/ftp_login` — credential brute-force using username & password lists

Example: run the ftp login brute-force module:

```
msf6 > use auxiliary/scanner/ftp/ftp_login
msf6 auxiliary(scanner/ftp/ftp_login) > set RHOSTS <target>
msf6 auxiliary(scanner/ftp/ftp_login) > set USER_FILE /usr/share/wordlists/dirb/common.txt    # or any user list
msf6 auxiliary(scanner/ftp/ftp_login) > set PASS_FILE /usr/share/wordlists/rockyou.txt      # or smaller lists for speed
msf6 auxiliary(scanner/ftp/ftp_login) > set THREADS 10
msf6 auxiliary(scanner/ftp/ftp_login) > run
```

Notes:
- Adjust `USER_FILE` and `PASS_FILE` paths to your available wordlists. Common locations: `/usr/share/wordlists/` and Metasploit's `data/wordlists`.
- Large wordlists can be noisy and slow; start small and escalate.

## 5) Brute-forcing with Hydra (alternative to Metasploit)

Hydra is fast and commonly used for FTP credential attacks:

```
hydra -L users.txt -P passwords.txt ftp://<target>

# example using a single username and a password list
hydra -l anonymous -P /usr/share/wordlists/rockyou.txt ftp://<target>
```

Hydra supports many protocols and can be tuned for concurrency. Use responsibly.

## 6) Using a found credential

Once you have valid credentials you can use a normal FTP client to connect and enumerate files:

```
ftp <target>
# then: username <user> and password <pass>

# or download/upload via curl
curl -u username:password ftp://<target>/path/to/file -O
```

If uploads are permitted, an attacker may drop web shells (if the FTP root maps to a web directory) or exfiltrate data.

## 7) Known vulnerable products and exploits

- vsftpd 2.3.4 backdoor (historical): had a one-byte backdoor that gave a shell when a specially crafted username was provided. Metasploit includes checks for it.
- ProFTPD mod_copy misconfiguration and other vendor-specific issues have had remotely exploitable flaws.
- Misconfiguration and weak credentials remain the most common risk.

Always verify whether a given version is actually vulnerable before attempting exploitation. Use `searchsploit`, Metasploit `search`, and CVE databases.

## 8) Post-exploitation and pivoting notes

- If the FTP server stores web content (e.g., /var/www/html), uploaded files may be executed by the web server — a common lateral attack path.
- Writable anonymous FTP can be used to host phishing pages or store stolen data.

## 9) Remediation and hardening

- Disable anonymous FTP unless explicitly required. If required, restrict it to read-only and chrooted directories.
- Use strong, unique passwords and disable clear-text authentication over untrusted networks; prefer SFTP/FTPS.
- Keep FTP server software up to date and remove old/backdoored versions.
- Limit access with firewall rules and use intrusion detection to monitor abnormal transfers or login attempts.

## 10) Quick checklist for FTP enumeration

- [ ] Run Nmap service/version scan with ftp NSE scripts
- [ ] Banner grab (nc / telnet) and identify version
- [ ] Check anonymous login and upload permissions
- [ ] Try Metasploit `ftp_login` or `anonymous` modules
- [ ] Use Hydra for high-speed credential attacks (if allowed)
- [ ] If credentials found, enumerate and check for writable directories mapping to web roots
- [ ] Search for product-specific exploits and verify applicability

## References

- Nmap NSE scripts (ftp-*) — https://nmap.org/nsedoc/
- Metasploit auxiliary modules — use `search type:auxiliary ftp` inside msfconsole
- OWASP, CVE and Exploit-DB for version-specific vulnerabilities

---

Revision: polished FTP enumeration notes with concrete commands, examples, and mitigation guidance.