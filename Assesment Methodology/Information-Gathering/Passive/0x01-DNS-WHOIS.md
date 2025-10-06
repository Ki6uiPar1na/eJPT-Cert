# 0x01 DNS & WHOIS (Passive)

Purpose: gather domain registration, ownership, and public DNS information without querying target hosts interactively beyond standard public lookups.

Commands and examples:

- whois
  - Usage: `whois example.com`
  - Example: `whois target.example` — returns registrar, registration/expiry dates, name servers, contact emails (may be redacted).

- host
  - Usage: `host target.example`
  - Example: `host example.com` — shows A/AAAA records and name servers.

- dig (DNS lookup utility)
  - Usage: `dig +short example.com ANY`
  - Example: `dig @8.8.8.8 example.com A` — query A record via Google DNS.

Tips:
- Use multiple public DNS servers (8.8.8.8, 1.1.1.1) to compare views.
- WHOIS info may be privacy-protected; check historical WHOIS via services if needed.
