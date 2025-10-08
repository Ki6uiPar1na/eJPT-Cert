# 0x01 — DNS & WHOIS (Passive)

Goal: gather registration and public DNS data to map ownership, hosting, mail servers, and public records.

Commands
- whois
  - Usage: `whois example.com`
  - What to look for: registrar, registrant contact (may be privacy-protected), registration/expiry dates, name servers.

- dig (recommended for structured DNS queries)
  - A record: `dig @8.8.8.8 example.com A +short`
  - MX records: `dig MX example.com +short`
  - NS records: `dig NS example.com +short`
  - TXT records: `dig TXT example.com +short`
  - Any/all: `dig example.com ANY +short` (note: some DNS servers rate-limit or ignore ANY)

- host
  - Quick: `host example.com`

- nslookup (interactive or one-liners)
  - `nslookup -type=mx example.com 8.8.8.8`

Useful web services
- crt.sh — certificate transparency lookup (find subdomains via TLS certs)
- dnsdumpster.com — visual DNS map
- securitytrails.com — historical DNS/WHOIS data

Tips
- Compare results across multiple public resolvers (8.8.8.8, 1.1.1.1, 9.9.9.9).
- Certificate transparency (crt.sh) is excellent for passive subdomain discovery.
- Always capture raw command output and record timestamps.

Quick scriptable examples
- List A records (bash):
  - `dig +short example.com A | sort -u`

- Reverse lookup for an IP (bash):
  - `dig -x 1.2.3.4 +short`
