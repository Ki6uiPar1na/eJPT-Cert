# 0x05 — Google Dorking

Google Dorking uses search engine operators to find specific content, exposed files, and configuration mistakes. It's a powerful passive reconnaissance method that often reveals information missed by automated scanners.

Important: always handle results responsibly. Finding sensitive data does not grant permission to access or exploit it.

Core operators (examples)
- site: — search within a host or domain
	- site:example.com
- filetype: — search by file extension
	- filetype:pdf
- inurl: / intitle: / intext: — search URL, title, or page text
	- inurl:admin
	- intitle:"index of"
	- intext:"password"
- cache: — view Google cached copy
	- cache:example.com
- ext: — alternate to filetype (older syntax)

Combining operators
- Find PDFs on a domain:
	- site:example.com filetype:pdf
- Find possible admin pages:
	- site:example.com inurl:(admin | login | wp-admin)
- Find exposed configuration or backup files:
	- site:example.com ("backup" | "sql" | "dump" ) filetype:(sql | bak | zip)

High-value dorks (curated)
- Exposed log files or databases:
	- filetype:log "index of" site:example.com
	- site:example.com filetype:sql | filetype:db
- Sensitive documents (PDF, XLS, DOC):
	- site:example.com filetype:pdf confidential | internal
	- site:example.com filetype:xls OR filetype:xlsx "password"
- Open directories (index of):
	- intitle:"index of" "parent directory" site:example.com
- Credentials & tokens (careful handling):
	- intext:password site:example.com
	- intext:"api_key" | intext:"secret" site:example.com

Google Hacking Database (GHDB)
- The GHDB (Exploit-DB) is a curated collection of dorks. Good source for inspiration and common patterns:
	- https://www.exploit-db.com/google-hacking-database

Safety, ethics and limitations
- Passive only: Dorking is non-interactive, but it can reveal sensitive files you must not access directly.
- Terms of service: scraping or mass-downloading results may violate Google TOS — keep queries manual and limited.
- False positives: results may include unrelated pages. Validate findings before treating them as sensitive.

Practical workflow
1. Define the information need (documents, admin panels, backups).
2. Build targeted dorks (restrict with site: and filetype: where possible).
3. Manually review top results and capture URLs (do not download sensitive files unless authorized).
4. Correlate with other passive sources (Wayback, crt.sh, DNS records).

Quick cheatsheet (examples)
- Find PDFs:
	- site:example.com filetype:pdf
- Find indexed admin paths:
	- site:example.com inurl:admin | inurl:login
- Find possible backups:
	- site:example.com (filetype:bak | filetype:sql | filetype:zip)
- Search within page titles
    -intitle:text
    
If you'd like, I can:
- add 10 curated dorks relevant to web apps or typical targets, or
- build a short script to run non-abusive, rate-limited dork queries against the Google Custom Search API (requires API key).


