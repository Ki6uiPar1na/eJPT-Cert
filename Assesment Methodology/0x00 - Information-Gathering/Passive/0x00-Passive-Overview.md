# 0x00 — Passive Information Gathering (Overview)

Passive recon collects public information without directly interacting with target infrastructure. It's low-noise, legally safer, and the best place to start.

Why start passive?
- Avoids generating alerts or leaving traces.
- Builds context: ownership, hosting, public assets, history, and personnel.
- Reduces the scope for later noisy scans.

Core passive categories
- DNS & WHOIS (registrar, name servers, MX/NS/TXT records)
- Website footprinting (robots.txt, sitemap.xml, security.txt, humans.txt)
- Archive/history (Wayback Machine)
- Technology detection (WhatWeb, Wappalyzer)
- OSINT (email harvesting, social profiles, breach data)
- Google Dorking (advanced search queries)

Recommended quick workflow
1. whois <domain>
2. dig @8.8.8.8 <domain> A MX NS TXT
3. curl --get https://<domain>/robots.txt
4. check sitemap: https://<domain>/sitemap.xml
5. Wayback / archive.org for historical pages
6. WhatWeb / Wappalyzer for tech fingerprinting
7. theHarvester / Google Dorks for emails and public files

Tools (examples)
- whois, dig, host, nslookup
- whatweb, wappalyzer (browser extension)
- theHarvester, GHDB (Google Hacking DB)
- dnsdumpster.com, securitytrails, crt.sh

Notes & ethics
- Passive does not eliminate legal considerations; respect rate limits and terms of service for services you query.
- Record sources and timestamps for every finding.
