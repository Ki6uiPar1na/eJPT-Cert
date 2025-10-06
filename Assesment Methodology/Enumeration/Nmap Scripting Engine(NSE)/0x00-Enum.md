## Enumeration — Practical Guide for eJPT Assessments

This document explains enumeration techniques and tools commonly used during penetration tests and Capture The Flag (CTF) exercises. It focuses on practical steps, commands, sample outputs, and interpretation to help you gather a target's surface information and discover attack vectors.

### Quick contract
- Inputs: IP address or hostname(s) discovered during discovery/recon phase.
- Outputs: A prioritized list of open ports, services, versions, misconfigurations, credentials, and reachable resources (shares, directories, APIs).
- Error modes: filtered/blocked ports, IDS/IPS noise, false positives in banners, rate-limited services.
- Success criteria: identify at least one exploitable service, valid credentials, or sensitive data exposure.

### High-level workflow
1. Confirm scope and rules of engagement (authorized IPs/ranges, allowed techniques, time windows).
2. Port and service scanning (TCP/UDP) to map the attack surface.
3. Service-specific enumeration (HTTP, SMB, LDAP, DNS, SNMP, databases, etc.).
4. Credential and configuration discovery (default creds, anonymous access, misconfigurations).
5. Validate and prioritize findings (exploitability, impact, evidence collection).

### Is enumeration active or passive?
- Passive: collecting information without directly interacting with the target (public DNS, Shodan, certificate transparency, OSINT). Low-noise and safe.
- Active: port scans, service probes, brute-force attempts, and other direct interactions. Higher noise and risk; ensure authorization.

### Port & service scanning
Nmap is the primary tool for active scanning. Save outputs for later parsing and reporting.

Recommended quick Nmap:

```bash
nmap -sC -sV -Pn --top-ports 100 -oA nmap/quick <target>
```

Full TCP port sweep (take more time):

```bash
nmap -p- -sV -Pn -T4 -oA nmap/allports <target>
```

UDP scan (slow/noisy):

```bash
nmap -sU -sV --top-ports 50 -Pn -T4 -oA nmap/udp <target>
```

Interpreting Nmap output:
- open = service reachable; follow up.
- filtered = packets dropped/filtered by firewall.
- service/version strings are useful for CVE lookups.

### Web (HTTP/HTTPS) enumeration
Goals: discover directories, parameters, virtual hosts, and possible vulnerabilities.

Dir discovery with gobuster:

```bash
gobuster dir -u http://<target> -w /usr/share/wordlists/dirb/common.txt -t 50 -o gobuster.txt
```

Virtual host discovery:

```bash
gobuster vhost -u http://<target> -w /path/to/vhosts.txt -t 40 -o vhosts.txt
```

Parameter fuzzing with ffuf:

```bash
ffuf -u http://<target>/FUZZ -w /usr/share/wordlists/param-mining.txt -t 50
```

Non-invasive scan:

```bash
nikto -host http://<target>
```

Manual inspection tips:
- Use a proxy (Burp Suite) to view headers, cookies, and hidden parameters.
- Look for 200 responses on uncommon paths, unusual headers, or error messages disclosing internals.

### SMB (Windows shares) enumeration
SMB often contains sensitive files and credentials.

List shares and connect (anonymous):

```bash
smbclient -L //<target> -N
smbclient //<target>/SHARE -N
```

Useful tools:
- enum4linux -a <target>
- smbmap -H <target>
- rpcclient -U '' <target>

What to look for:
- Writable shares, backup files (.bak, .sql, .zip), scripts with credentials, and user lists.

### LDAP / Active Directory
LDAP can leak users, groups, SPNs, and domain configuration.

Anonymous bind example:

```bash
ldapsearch -x -h <target> -b "dc=example,dc=com"
```

Authenticated enumeration (when you have creds):

```bash
ldapsearch -x -D "cn=admin,dc=example,dc=com" -W -b "dc=example,dc=com"
```

Collect: sAMAccountName, userPrincipalName, memberOf, servicePrincipalName.

### DNS enumeration
DNS can reveal subdomains, mail servers, and internal hostnames.

Zone transfer attempt:

```bash
dig AXFR @ns1.example.com example.com
```

Subdomain discovery tools: `amass`, `subfinder`, `assetfinder`.

Be wary of wildcard DNS which creates noisy false positives.

### SNMP
Check for default community strings:

```bash
snmpwalk -v2c -c public <target> system
```

Full walk (may be large):

```bash
snmpwalk -v2c -c public <target> .1
```

Look for device configs, installed software, or plaintext secrets.

### Databases and other services
Try connecting with default/null credentials and test for unauthenticated access.

Examples:
- MySQL: `mysql -h <target> -P 3306 -u root -p` (try blank password)
- Redis: `redis-cli -h <target> -p 6379` then `INFO` or `CONFIG GET dir`
- Elasticsearch/MongoDB: try HTTP curl or native clients to list indices/collections.

### Credentials harvesting
Search for secrets in discovered files: config files, backups, and scripts.

Grep example:

```bash
grep -R --line-number -i "password\|pass\|secret\|api_key\|aws_access_key_id" ./downloaded_files
```

Also check for tokens in web pages, JavaScript files, and cookies.

### Automation & OSINT
- Passive: Shodan, Censys, Certificate Transparency, GitHub search for keys.
- Active: NSE scripts, automated scanners, and chaining lightweight tools (nmap -> enum -> gobuster).

### Example mini workflow
1. Quick nmap: `nmap -sC -sV -Pn --top-ports 100 -oA nmap/quick <target>`
2. If web ports found: run gobuster/ffuf and nikto; inspect in a proxy.
3. If SMB found: run enum4linux, smbmap, then download interesting files.
4. If LDAP/AD: ldapsearch for user lists and SPNs.

### Evidence & reporting
- Save raw outputs and screenshots.
- For each finding include: reproduction steps, exact commands, output snippets, and impact assessment.

### Prioritization
- High: Active credentials, RCE, data exfiltration, domain admin artifacts.
- Medium: Sensitive files, writable backups, outdated software with public exploits.
- Low: Info disclosure (versions, hostnames) unless it enables further attacks.

### Quick checklist
- [ ] Confirm scope and authorization
- [ ] Run quick TCP/UDP scan and save results
- [ ] Enumerate web services (gobuster, ffuf) and manual inspection
- [ ] Enumerate SMB/LDAP/DNS/SNMP as applicable
- [ ] Collect and search files for credentials
- [ ] Document and prioritize findings with evidence

### Defensive and edge considerations
- Use lower timing templates (-T2/-T3) to avoid IDS/IPS triggers.
- Respect account lockout policies when testing credentials.
- WAFs and proxies may block fuzzers — use authenticated testing paths or bypass techniques if authorized.

## Notes
This file is a living guide — tailor tools, wordlists, and scan intensity to each engagement and always follow the rules of engagement.

--
Revision: comprehensive enumeration guide for practical assessments.


