# 0x06 — DNS Enumeration

DNS enumeration is a high-value passive step in recon: it reveals public-facing hosts, mail infrastructure, authoritative name servers, and metadata (SPF/DKIM) that help scope an engagement.

Always record raw output and timestamps. Use passive sources first (crt.sh, DNS history) and only run active or noisy checks (AXFR, repeated brute force) with permission.

What to collect

- A / AAAA — host IPs
- MX — mail exchangers
- NS — authoritative name servers
- TXT — SPF, DKIM, verification tokens
- SOA — zone authority and serial (useful for detecting secondary name servers)
- CNAME — alias chains (redirect to another domain)
- PTR — reverse DNS for IPs

Essential tools & one-liners

- dig (recommended for scripting)
  - A: `dig +short A example.com`
  - AAAA: `dig +short AAAA example.com`
  - MX: `dig +short MX example.com`
  - NS: `dig +short NS example.com`
  - TXT: `dig +short TXT example.com`
  - SOA: `dig +short SOA example.com`

- host
  - Full info: `host -a example.com`

- nslookup (interactive):
  - `nslookup` then `server 8.8.8.8` and `set q=mx` etc.

- dnsrecon / dnsenum (automation)
  - `dnsrecon -d example.com`
  - `dnsenum --enum example.com`

- Passive web tools
  - crt.sh (certificate transparency) — find subdomains via TLS certs
  - dnsdumpster.com — visual DNS mapping
  - securitytrails.com — historical DNS and replication data

Certificate Transparency (crt.sh) and CT logs

Use `crt.sh?q=%25.example.com` (or the web UI) to discover hostnames that appear in issued TLS certificates. This is often the best passive source for subdomains forgotten by DNS.

Active vs passive sources

- Passive: crt.sh, securitytrails, DNS history, certificate logs — no direct queries to target name servers.
- Active: `dig` lookups and AXFR attempts against the target's name servers — may be logged.

AXFR (Zone Transfer) — handle with extreme caution

Zone transfers (AXFR) can disclose the entire DNS zone. They are active and often logged by providers. Only attempt when authorized.

- Check authoritative NS first:
  - `dig +short NS example.com`
- Test AXFR (example):
  - `dig AXFR example.com @ns1.example.com`
- Use dnsrecon/dnsenum wrappers:
  - `dnsrecon -d example.com -t axfr`
  - `dnsenum --axfr example.com`

If AXFR succeeds, treat results as sensitive and proceed according to scope (do not publish).

Subdomain discovery (recommended sequence)

1) Passive collection: crt.sh, securitytrails, archived DNS.
2) Passive tools: `subfinder -d example.com`, `amass enum -passive -d example.com`.
3) Active DNS resolution of candidates: `massdns` or `dnsx` to validate quickly.
4) Brute-force (only if permitted): `gobuster dns` / `ffuf` / `massdns` with curated wordlists.

Example automation (bash)

Collect A records then reverse lookup:

```bash
dig +short A example.com | sort -u > a_records.txt
while read ip; do dig -x "$ip" +short >> ptrs.txt; done < a_records.txt
sort -u ptrs.txt
```

Resolve candidate subdomains (fast validation using dnsx):

```bash
cat candidates.txt | dnsx -resp -silent -a -ip -cname -ttl -o resolved.txt
```

Quick detection of exposed TXT/SPF/DKIM values

```bash
dig +short TXT example.com | sed 's/"//g'
```

Notes & tips

- Use multiple resolvers (8.8.8.8, 1.1.1.1) to compare answers when debugging.
- Certificate transparency logs often reveal subdomains that are not present in current DNS but were issued in certs.
- Aggregate and de-duplicate all outputs into a single CSV/JSON asset list for downstream tasks.
- Respect rate limits and legal boundaries — DNS servers are frequently monitored by abuse teams.

Further reading

- RFCs: basic DNS (RFC 1034/1035) and zone transfer behavior.
- Tools: dig (man dig), dnsrecon, dnsenum, massdns, dnsx, amass, subfinder.