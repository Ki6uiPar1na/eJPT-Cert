
# 0x01 Nmap - Host Discovery

Purpose
-------
Discover which hosts are alive on a network or which IPs resolve for a domain. Host discovery is the first active step in network reconnaissance and helps you focus subsequent port/service scans only on live targets.

How to read this file
---------------------
For each command you'll find:
- Command: the exact nmap (or related) command to run.
- What it does: concise behavior summary.
- Why use it: when this command is the right choice.
- Why not: limitations, false-positives, or why it might fail.
- Flags explained: breakdown of the important options used.

1) Simple ICMP ping sweep (default ping-based discovery)
-------------------------------------------------------
Command:
```
nmap -sn -PE -T4 192.168.1.0/24
```
What it does:
- Sends ICMP echo requests (ping) to hosts in the specified subnet and reports hosts that respond.

Why use it:
- Fast and low-effort way to detect live hosts on a local network where ICMP is allowed.

Why not:
- Many hosts, routers, or firewalls block ICMP. If ICMP is filtered you'll miss live hosts (false negatives).

Flags explained:
- `-sn` : ping scan only — do not perform port scans.
- `-PE` : use ICMP Echo Request packets for discovery.
- `-T4` : timing template for faster scanning (use with caution on unstable networks).

Edge cases & tips:
- If you see few results, follow up with TCP/ARP discovery methods (below). Use `-T2`/`-T3` for slower networks.

2) SYN ping to specific TCP ports (works when ICMP is blocked)
----------------------------------------------------------------
Command:
```
nmap -sn -PS22,80,443 10.0.0.0/24
```
What it does:
- Sends TCP SYN probes to the specified ports on each host; if a SYN/ACK is received the host is considered up.

Why use it:
- Useful when ICMP is filtered but common TCP ports are reachable. Good for discovering hosts that run SSH/HTTP services.

Why not:
- Can trigger firewall/IDS rules. If the target actively blocks these probes you'll still get false negatives. Requires more care (and authorization).

Flags explained:
- `-PS22,80,443` : TCP SYN ping to ports 22, 80 and 443.
- `-sn` : ping scan only (no port/service enumeration after discovery).

Notes:
- Choose probe ports thoughtfully; pick ports likely to be open in the environment being tested.

3) ARP discovery for local Ethernet networks (most reliable on LAN)
----------------------------------------------------------------
Command:
```
nmap -sn -PR 192.168.1.0/24
```
What it does:
- Uses ARP requests to discover hosts on the same Ethernet segment. ARP cannot be filtered by host-level firewalls because it's a layer 2 protocol.

Why use it:
- Almost always the fastest and most reliable method to discover hosts on a local network.

Why not:
- Works only on the local LAN segment (won't work across routers/VLANs). Requires that you're on the same broadcast domain.

Flags explained:
- `-PR` : ARP ping.
- `-sn` : ping-only scan (no port scan).

4) Treat hosts as up (skip host discovery) — target specified IPs directly
---------------------------------------------------------------------
Command:
```
nmap -Pn -p- 10.0.0.5
```
What it does:
- Skips host discovery entirely and treats specified hosts as up; then performs a full TCP port scan (`-p-` scans all ports).

Why use it:
- Use when host discovery is unreliable (ICMP/TCP/ARP are blocked), or when you already know an IP is live and want a full port sweep.

Why not:
- This increases scan time because it won't rule out dead hosts. Also more likely to be noisy and detected.

Flags explained:
- `-Pn` : no host discovery (treat hosts as up).
- `-p-` : scan all ports (1-65535).

Privilege note:
- Some scan types or options (like `-sS` SYN scans or OS detection `-O`) may require root/administrator privileges to send raw packets.

5) Fast ARP / passive LAN discovery utility
-----------------------------------------
Command:
```
netdiscover -r 192.168.1.0/24
```
What it does:
- Passive/active ARP scanner that listens for ARP replies on the LAN and can send ARP probes; useful for quick mapping of hosts and MAC addresses.

Why use it:
- Extremely fast on a local network and gives MAC-vendor information (helps identify routers, printers, IoT devices).

Why not:
- Only works on local LAN. It may be too noisy for some environments and is not useful for remote targets.

Install tip:
- `sudo apt install netdiscover` or get it from the distro package manager.

6) Combination workflows and recommended sequences
-------------------------------------------------
- Passive first: collect DNS/A records, ARP caches, and public DNS before any active scans.
- Start with ARP on your LAN (`-PR`) if local.
- If remote or ICMP filtered, use TCP SYN pings (`-PS`) to likely ports.
- If discovery fails but you know the IPs, use `-Pn` followed by targeted `-p` scans.

Common pitfalls and troubleshooting
----------------------------------
- False negatives: firewalls and host-based filters often drop ICMP/TCP probes — try multiple discovery techniques.
- False positives: NAT devices and load balancers can respond and cause multiple hosts to appear up.
- Legal/ethical: active host discovery is noisy. Only scan networks you are authorized to test.

Short cheat sheet (copy/paste)
------------------------------
ICMP ping sweep (fast, quiet when allowed):
```
nmap -sn -PE -T4 192.168.1.0/24
```
SYN ping if ICMP blocked:
```
nmap -sn -PS22,80,443 10.0.0.0/24
```
ARP discovery (local LAN — reliable):
```
nmap -sn -PR 192.168.1.0/24
```
Skip discovery and run full TCP port scan on known IP:
```
nmap -Pn -p- 10.0.0.5
```

References and further reading
------------------------------
- `man nmap` and `nmap --help` for up-to-date options.
- Nmap book/online docs: https://nmap.org/book/host-discovery.html

