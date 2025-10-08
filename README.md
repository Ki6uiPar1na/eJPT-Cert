# eJPT Study Notes

This repository contains structured study notes and quick command references for the eJPT certification. The content is organized as numbered modules: 0x00 covers Information Gathering (passive + active recon), 0x01 covers Enumeration, and so on.

Below is a polished, detailed overview of the 0x00 — Information Gathering module with direct links to the detailed notes in the repo.

## 0x00 — Information Gathering (Overview)

Information gathering is the foundation of any penetration test. It is split into two complementary approaches:

- Passive: collect publicly available data without touching the target directly (WHOIS, public DNS, archives, OSINT).
- Active: interact with the target (scans, DNS zone transfer attempts, subdomain bruteforce). Active techniques are noisy and require explicit authorization.

Read the full module: [0x00 README](Assesment%20Methodology/0x00%20-%20Information-Gathering/README.md)

### Passive Recon — what to do first

Start with passive recon to build context and reduce noisy scans later.

- DNS & WHOIS
	- File: [DNS & WHOIS](Assesment%20Methodology/0x00%20-%20Information-Gathering/Passive/0x01-DNS-WHOIS.md)
	- Quick commands:
		- whois: `whois example.com`
		- dig A record: `dig @8.8.8.8 example.com A`
		- list MX: `dig MX example.com`

- Website footprinting (robots, sitemap, security.txt, humans.txt)
	- File: [Website Footprinting](Assesment%20Methodology/0x00%20-%20Information-Gathering/Passive/0x02-Website-Footprinting.md)
	- Quick commands/examples:
		- fetch robots: `curl --get https://example.com/robots.txt`
		- fetch sitemap: `curl --get https://example.com/sitemap.xml`
		- mirror site (offline): `httrack "https://example.com" -O ./example_mirror`

- Technology detection
	- File: [Tech Detection](Assesment%20Methodology/0x00%20-%20Information-Gathering/Passive/0x03-Tech-Detection.md)
	- Tools: `whatweb`, `wappalyzer` (browser extension)
	- Example: `whatweb https://example.com`

- OSINT: email harvesting & social media
	- File: [OSINT - Email & Social](Assesment%20Methodology/0x00%20-%20Information-Gathering/Passive/0x04-OSINT-Email-Social.md)
	- Tools: `theHarvester`, `HaveIBeenPwned` (web/API)
	- Example: `theharvester -d example.com -l 200 -b google`

- Google Dorking (advanced web searches)
	- File: [Google Dorking](Assesment%20Methodology/0x00%20-%20Information-Gathering/Passive/0x05-Google%20Dorking.md)
	- Example dork: `site:example.com filetype:pdf` or `intitle:"index of" site:example.com`

- DNS-specific passive techniques and tools
	- File: [DNS Enumeration](Assesment%20Methodology/0x00%20-%20Information-Gathering/Passive/0x06-DNS%20Enum.md)
	- Web tool: https://dnsdumpster.com/
	- CLI: `dig`, `host`, `nslookup`, `dnsenum`, `dnsrecon`

### Active Recon — when authorized

Active recon lets you find hosts, ports and services but is noisy. Only do this with permission.

- Active overview
	- File: [Active Overview](Assesment%20Methodology/0x00%20-%20Information-Gathering/Active/0x00-Active-Overview.md)

- Nmap — host discovery & scanning
	- File: [Nmap Host Discovery](Assesment%20Methodology/0x00%20-%20Information-Gathering/Active/0x01-Nmap-Host-Discovery.md)
	- Useful commands:
		- ICMP ping sweep: `nmap -sn -PE -T4 192.168.1.0/24`
		- SYN ping (when ICMP filtered): `nmap -sn -PS22,80,443 10.0.0.0/24`
		- ARP discovery (local LAN): `nmap -sn -PR 192.168.1.0/24`
		- Treat host as up and full port scan: `nmap -Pn -p- 10.0.0.5`

- DNS Zone Transfers
	- File: [DNS Zone Transfers](Assesment%20Methodology/0x00%20-%20Information-Gathering/Active/0x02-DNS-Zone-Transfers.md)
	- Test with dig: `dig AXFR example.com @ns1.example.com`
	- Tools: `dnsrecon -d example.com -t axfr`, `dnsenum --axfr example.com`

- Subdomain enumeration (bruteforce & virtual hosts)
	- File: [Subdomain Enumeration](Assesment%20Methodology/0x00%20-%20Information-Gathering/Active/0x03-subdomain%20Enum.md)
	- Tools & examples:
		- gobuster DNS mode: `gobuster dns -d example.com -w /path/to/wordlist`
		- gobuster vhost mode: `gobuster vhost -u https://example.com -w /path/to/wordlist --exclude-length 1542`
		- amass/subfinder for passive+active enumeration

### Quick workflow (recommended)

1. Passive recon: WHOIS, dig, DNSDumpster, robots.txt, sitemap, theHarvester, WhatWeb.
2. Subdomain enumeration (passive tools → bruteforce if needed).
3. Host discovery (Nmap ARP/TCP/ICMP as appropriate).
4. Targeted service enumeration & version detection.
5. Document findings and proceed to safe exploitation/validation steps with authorization.

### Links (full module files)

- [Assesment Methodology root folder](Assesment%20Methodology/)
- 0x00 — Information Gathering README: [link](Assesment%20Methodology/0x00%20-%20Information-Gathering/README.md)
- Passive folder: [link](Assesment%20Methodology/0x00%20-%20Information-Gathering/Passive/)
	- DNS & WHOIS: [link](Assesment%20Methodology/0x00%20-%20Information-Gathering/Passive/0x01-DNS-WHOIS.md)
	- Website Footprinting: [link](Assesment%20Methodology/0x00%20-%20Information-Gathering/Passive/0x02-Website-Footprinting.md)
	- Tech Detection: [link](Assesment%20Methodology/0x00%20-%20Information-Gathering/Passive/0x03-Tech-Detection.md)
	- OSINT: [link](Assesment%20Methodology/0x00%20-%20Information-Gathering/Passive/0x04-OSINT-Email-Social.md)
	- Google Dorks: [link](Assesment%20Methodology/0x00%20-%20Information-Gathering/Passive/0x05-Google%20Dorking.md)
	- DNS Enum: [link](Assesment%20Methodology/0x00%20-%20Information-Gathering/Passive/0x06-DNS%20Enum.md)
- Active folder: [link](Assesment%20Methodology/0x00%20-%20Information-Gathering/Active/)
	- Active Overview: [link](Assesment%20Methodology/0x00%20-%20Information-Gathering/Active/0x00-Active-Overview.md)
	- Nmap Host Discovery: [link](Assesment%20Methodology/0x00%20-%20Information-Gathering/Active/0x01-Nmap-Host-Discovery.md)
	- DNS Zone Transfers: [link](Assesment%20Methodology/0x00%20-%20Information-Gathering/Active/0x02-DNS-Zone-Transfers.md)
	- Subdomain Enumeration: [link](Assesment%20Methodology/0x00%20-%20Information-Gathering/Active/0x03-subdomain%20Enum.md)

