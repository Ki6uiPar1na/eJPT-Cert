# 0x02 — DNS Zone Transfers (AXFR)

Goal: test authoritative name servers for misconfigured zone transfers (AXFR) which can leak the entire zone.

Check authoritative NS records
- `dig NS example.com +short`

Test AXFR with dig (active; only run with permission)
- `dig AXFR example.com @ns1.example.com`
- If successful, you'll receive all the DNS records in the zone.

Tools
- dnsrecon: `dnsrecon -d example.com -t axfr`
- dnsenum: `dnsenum --axfr example.com`

Analysis
- If AXFR succeeds, extract hostnames and IPs and validate whether any are in scope for further scanning.
- Treat AXFR results as sensitive — they often contain internal hostnames and subdomains.

Safety
- AXFR is an active check and may be logged. Only run against targets you are authorized to test.
