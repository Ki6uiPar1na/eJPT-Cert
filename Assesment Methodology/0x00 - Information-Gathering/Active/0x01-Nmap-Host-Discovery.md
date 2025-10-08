# 0x01 — Nmap Host Discovery

Goal: reliably discover live hosts before port/service scans.

Common techniques

1) ICMP ping sweep (fast when ICMP allowed)
- `nmap -sn -PE -T4 192.168.1.0/24`
- Notes: may miss hosts if ICMP is blocked.

2) TCP SYN ping to common ports (works if ICMP blocked)
- `nmap -sn -PS22,80,443 10.0.0.0/24`
- Notes: can trigger IDS/IPS; choose ports appropriate for the environment.

3) ARP discovery (best for local LAN)
- `nmap -sn -PR 192.168.1.0/24`
- Notes: ARP is reliable but only works on the same broadcast domain.

4) Treat hosts as up and scan ports directly (when discovery fails or you know IPs)
- `nmap -Pn -p- 10.0.0.5`
- Notes: slower and noisier; `-Pn` disables host discovery.

Practical tips
- Use `-oA <prefix>` to save Nmap results in all formats:
  - `nmap -sV -oA scan_results 10.0.0.5`
- For large ranges, consider `masscan` for fast port discovery, then follow up with Nmap.
- Run with root for SYN scans (`-sS`) and OS detection (`-O`) when needed.

Cheat sheet
- ICMP sweep: `nmap -sn -PE -T4 192.168.1.0/24`
- SYN ping: `nmap -sn -PS22,80,443 10.0.0.0/24`
- ARP: `nmap -sn -PR 192.168.1.0/24`
- Full port scan: `nmap -Pn -p- 10.0.0.5`

