# 0x00 — Active Information Gathering (Overview)

Active recon interacts directly with target systems and is noisy. Only perform these actions with explicit authorization and clear scope.

Common active tasks
- Host discovery (ICMP/TCP/ARP)
- Port scanning and service/version detection
- DNS zone transfer attempts (AXFR)
- Subdomain brute-force and virtual host (vhost) enumeration

Safety rules
- Obtain written permission and scope before any active scans.
- Use low rate limits and document timings.
- Avoid destructive tests during discovery.

Recommended sequence
1. Passive recon (already completed).
2. Subdomain enumeration (passive → active bruteforce when needed).
3. Host discovery and targeted port scans (start with non-intrusive probes).
4. Service enumeration and version detection for identified targets.
