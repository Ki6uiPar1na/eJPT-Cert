# Web Server Enumeration — polished guide

This document covers focused enumeration techniques for HTTP(S) web servers, preserves your demo workflow (target: `demo.ine.local`), and adds additional commands, descriptions, and troubleshooting tips.

Important: only test systems you own or have explicit permission to test.

---

## What is a web server?
A web server is software that serves HTTP/HTTPS requests (pages, APIs, files) to clients. Popular implementations include:

- Apache HTTP Server (httpd)
- Nginx
- Microsoft IIS
- Lighttpd
- Caddy

HTTP vs HTTPS
- HTTP: plaintext protocol on port 80 (commonly).
- HTTPS: HTTP over TLS on port 443; encryption hides payloads but headers and certificate info are visible.

---

## Demo (your flow preserved)
Target: demo.ine.local

High-level plan
1. Prepare Metasploit (database + workspace).
2. Use Metasploit auxiliary http modules for quick discovery (version, headers, robots).
3. Fetch `robots.txt` and `sitemap.xml` and manually curl potential directories.
4. Use a directory scanner (brute-force) such as `dir_scanner` in Metasploit or external tools (`gobuster`, `ffuf`, `dirsearch`).
5. If authentication is required, use `http_login` in Metasploit or targeted external brute force tools with web-specific wordlists.

### Metasploit demo steps (concise)

```text
# Start PostgreSQL (if required by your Metasploit install)
sudo service postgresql start

# Start msfconsole
msfconsole

# Create and switch to a workspace
msf6 > workspace -a web_enum
msf6 > workspace web_enum

# (optionally) set a global RHOSTS value for convenience
msf6 > setg RHOSTS demo.ine.local
```

Search available HTTP auxiliary modules and use them:

```text
msf6 > search type:auxiliary name:http

# Example: detect HTTP server/version
msf6 > use auxiliary/scanner/http/http_version
msf6 auxiliary(http_version) > show options
msf6 auxiliary(http_version) > set RHOSTS demo.ine.local
msf6 auxiliary(http_version) > run
```

`http_version` output typically shows server header and hints (e.g., `Apache/2.4.41 (Ubuntu)`).

Next: enumerate headers and robots

```text
# robots.txt
msf6 > search type:auxiliary name:robots
msf6 > use auxiliary/scanner/http/robots_txt
msf6 auxiliary(robots_txt) > set RHOSTS demo.ine.local
msf6 auxiliary(robots_txt) > run

# directory bruteforce (Metasploit dir_scanner)
msf6 > use auxiliary/scanner/http/dir_scanner
msf6 auxiliary(dir_scanner) > set RHOSTS demo.ine.local
msf6 auxiliary(dir_scanner) > set THREADS 10
msf6 auxiliary(dir_scanner) > set WORDLIST /usr/share/wordlists/dirb/common.txt
msf6 auxiliary(dir_scanner) > run
```

If `http_login` is required for login brute-force:

```text
msf6 > use auxiliary/scanner/http/http_login
msf6 auxiliary(http_login) > show options
msf6 auxiliary(http_login) > set RHOSTS demo.ine.local
msf6 auxiliary(http_login) > set USER_FILE /root/users.txt
msf6 auxiliary(http_login) > set PASS_FILE /root/passwords.txt
msf6 auxiliary(http_login) > set STOP_ON_SUCCESS true
msf6 auxiliary(http_login) > set THREADS 5
msf6 auxiliary(http_login) > run
```

---

## Practical commands outside Metasploit (recommended)
These tools are fast and often easier for web-specific tasks.

### 1) Recon & version discovery

```bash
# Basic port/service scan
nmap -sV -p80,443 -Pn demo.ine.local

# Web-focused NSE scripts
nmap -p80,443 --script http-methods,http-title,http-server-header,safe -Pn demo.ine.local

# Quick banner/header check
curl -I -s -S http://demo.ine.local
curl -I -s -S https://demo.ine.local

# TLS certificate details
openssl s_client -connect demo.ine.local:443 -servername demo.ine.local </dev/null 2>/dev/null | openssl x509 -noout -text | sed -n '/Subject:/, /X509v3/p'
```

Expected `curl -I` output (example):

```
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Wed, 08 Oct 2025 12:00:00 GMT
Content-Type: text/html; charset=UTF-8
```

### 2) Fetch robots.txt and sitemap.xml

```bash
curl -fsSL https://demo.ine.local/robots.txt -o robots.txt || echo 'no robots'
curl -fsSL https://demo.ine.local/sitemap.xml -o sitemap.xml || echo 'no sitemap'

# Quick manual inspection
cat robots.txt
```

`robots.txt` often contains disallowed paths that are good candidates for manual inspection.

### 3) Directory discovery (brute/wordlists)

Use wordlists from SecLists (`/usr/share/seclists/Discovery/Web-Content/`).

```bash
# gobuster (fast and common)
gobuster dir -u http://demo.ine.local -w /usr/share/seclists/Discovery/Web-Content/common.txt -t 50 -x php,html,txt -o gobuster-results.txt

# ffuf (fuzzing tool)
ffuf -u http://demo.ine.local/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -mc 200,301,302 -o ffuf.json

# dirsearch (python)
python3 /usr/share/dirsearch/dirsearch.py -u http://demo.ine.local -e php,html,txt -w /usr/share/seclists/Discovery/Web-Content/common.txt -t 20
```

Tips
- Use `-x`/extensions to find script endpoints and pages.
- Save results and follow up on HTTP 200 / 301 / 302 / 401 responses.

### 4) Vulnerability scanners

```bash
# nikto (basic/vulnerability scans)
nikto -h http://demo.ine.local -output nikto-report.txt

# WhatWeb for technology detection
whatweb -v demo.ine.local
```

---

## Manual exploration & Burp
- Configure your browser to proxy through Burp Suite and explore the site.
- Intercept requests to examine hidden parameters, cookies, and auth flows.
- Use Repeater to test payloads and Intruder for targeted fuzzing.

---

## Authenticated flows and credential testing
- Prefer web-aware tools (Hydra, Burp Intruder, Metasploit `http_login`) and small lists to avoid lockouts.
- Use valid username/password combos when available; prefer contextual wordlists (e.g., `company-usernames.txt`).

Example Hydra command (use carefully):

```bash
hydra -L users.txt -P passwords.txt demo.ine.local http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect" -t 4 -w 5
```

Note: Adjust the form arguments and failure string to match the target.

---

## Post-discovery: CMS, hidden endpoints, and content analysis
- Retrieve site maps, check `/wp-admin`, `/admin`, `/login`, and common CMS paths.
- Use `wpscan` for WordPress or specialized CMS tools when appropriate.
- Use `waybackurls`, `gau` (GetAllUrls), and `meg` to gather historical endpoints from archive sources.

Example wayback/ga usage (if installed):

```bash
echo demo.ine.local | waybackurls | grep -E "(login|admin|wp-admin|/upload)" > interesting-urls.txt
```

---

## Example workflow (complete, step-by-step)
1. DNS/IP + initial scan

```bash
host demo.ine.local
nmap -sV -p80,443 -Pn demo.ine.local
```

2. Quick header & TLS check

```bash
curl -I https://demo.ine.local
openssl s_client -connect demo.ine.local:443 -servername demo.ine.local </dev/null | openssl x509 -noout -text | sed -n '/Subject:/, /X509v3/p'
```

3. Robots, sitemap, manual link checks

```bash
curl -fsSL https://demo.ine.local/robots.txt | sed -n '1,200p'
curl -fsSL https://demo.ine.local/sitemap.xml | sed -n '1,200p'
```

4. Directory discovery

```bash
gobuster dir -u http://demo.ine.local -w /usr/share/seclists/Discovery/Web-Content/common.txt -t 50 -x php,html,txt
```

5. If login discovered, test authentication

```bash
# Metasploit http_login (or use Hydra/Burp Intruder)
msf6 > use auxiliary/scanner/http/http_login
msf6 auxiliary(http_login) > set RHOSTS demo.ine.local
msf6 auxiliary(http_login) > set USER_FILE /root/users.txt
msf6 auxiliary(http_login) > set PASS_FILE /root/passwords.txt
msf6 auxiliary(http_login) > set STOP_ON_SUCCESS true
msf6 auxiliary(http_login) > run
```

---

## Notes & troubleshooting
- If the site uses many virtual hosts, test host headers (e.g., `curl -H 'Host: vhost.example.com' http://<ip>/`).
- If TLS errors occur, inspect certificates and try specifying `-k` in `curl` for testing (beware of MITM implications).
- Some sites block scanners—throttle your requests and use `--delay` or lower `-t`/`-T` values.
- Keep evidence (screenshots, saved pages, request/response pairs) for reporting.

---

## References & wordlists
- SecLists (Discovery/Web-Content)
- OWASP Testing Guide (web enumeration sections)
- Metasploit auxiliary module docs (within msfconsole: `info <module>`)

---

If you want, I can:
- Add a ready-to-run `recon-web.sh` script that runs the safe `nmap`, `curl`, `nmblookup`/`http_version` checks and saves output to `loot/web/demo.ine.local/`.
- Add sample `users.txt` and `passwords.txt` templates (small, safe lists) under `tools/`.

Tell me which and I will create the files and commit them to the repo.
