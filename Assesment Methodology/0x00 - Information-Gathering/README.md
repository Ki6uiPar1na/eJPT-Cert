# 0x00 — Information Gathering

This directory contains concise, practical notes and commands for passive and active reconnaissance. Files are numbered (0x00...) and organized into `Passive/` and `Active/` subfolders. Use passive techniques first to build context; proceed to active scans only with explicit authorization.

Contents (quick links):

- Passive/
  - `0x00-Passive-Overview.md` — why passive first and common tools
  - `0x01-DNS-WHOIS.md` — whois, dig, host, nslookup examples
  - `0x02-Website-Footprinting.md` — robots, sitemap, Wayback, meta files
  - `0x03-Tech-Detection.md` — WhatWeb, Wappalyzer, header inspection
  - `0x04-OSINT-Email-Social.md` — theHarvester, HIBP, social search
  - `0x05-Google Dorking.md` — useful dorks and safety notes
  - `0x06-DNS Enum.md` — DNS tools and workflows

- Active/
  - `0x00-Active-Overview.md` — active recon principles and safety
  - `0x01-Nmap-Host-Discovery.md` — host discovery cheat sheet
  - `0x02-DNS-Zone-Transfers.md` — AXFR testing and tools
  - `0x03-subdomain Enum.md` — subdomain & vhost enumeration

How to use these notes

1. Passive first: collect public DNS, WHOIS, robots/sitemap, Wayback, and OSINT data.
2. Subdomain enumeration: passive tools then targeted brute-force where needed.
3. Host discovery: Nmap ARP/TCP/ICMP as appropriate.
4. Targeted service enumeration and version detection with permission.

Want extras?
- I can auto-add 1–2 line summaries for each file from their intros.
- I can percent-encode links or rename files to remove spaces and update links.
- I can produce a printable cheat-sheet PDF with core commands.



