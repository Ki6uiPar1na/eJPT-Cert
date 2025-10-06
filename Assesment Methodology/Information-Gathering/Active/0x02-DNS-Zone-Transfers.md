# 0x06 DNS Zone Transfers and Risks

Purpose: check whether authoritative name servers allow zone transfers (AXFR) which can disclose all DNS records.

Commands:

- Test AXFR with dig
  - `dig AXFR example.com @ns1.example.com`

- Use dnsrecon
  - `dnsrecon -d example.com -t axfr`

- Use dnsenum
  - `dnsenum --axfr example.com`

If successful, you'll receive all hostnames and IPs managed in that DNS zone — treat results as sensitive.
