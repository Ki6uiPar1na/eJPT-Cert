# Nmap — Practical Reference

This file contains useful Nmap commands and short explanations for common scanning tasks during assessments. Use `sudo` when a scan requires raw packet privileges (SYN scans, OS detection, some NSE scripts).

## Basic scans

- Default quick scan (shows common ports):

```bash
nmap <target>
```

- Skip host discovery (treat hosts as up) — useful when ICMP is blocked:

```bash
nmap -Pn <target>
```

## Port selection

- Scan all TCP ports (0-65535):

```bash
nmap -p- -T4 <target>
```

- Top N ports (faster, common services):

```bash
nmap --top-ports 100 -T4 <target>
```

## Scan types

- TCP SYN (stealth) scan - requires root for raw packets:

```bash
sudo nmap -sS <target>
```

- TCP connect scan (no raw sockets required):

```bash
nmap -sT <target>
```

- UDP scan (slow and noisy):

```bash
sudo nmap -sU --top-ports 50 <target>
```

## Service & OS detection

- Service/version detection and OS fingerprinting (aggressive):

```bash
sudo nmap -sV -O <target>
```

- Aggressive mode (combines scripts, version, OS detection, traceroute):

```bash
sudo nmap -A <target>
```

## Timing and stealth

- Use timing templates to make scans faster or quieter: `-T0` (paranoid) ... `-T5` (insane).
- Reduce rate to avoid IDS/IPS: `--scan-delay 200ms` or `--max-rate 10`.

## Output formats

- Normal text: `-oN result.txt`
- XML: `-oX result.xml` (useful for import into tools)
- Grepable (deprecated but sometimes handy): `-oG result.gnmap`
- All formats at once: `-oA basename` (creates `basename.nmap`, `.xml`, `.gnmap`)

```bash
sudo nmap -sC -sV -oA nmap/scan1 <target>
```

## NSE (Nmap Scripting Engine)

- Run default safe scripts: `-sC` (equivalent to `--script=default`).
- Run specific script or category, e.g., HTTP enumeration or vulnerability checks:

```bash
sudo nmap --script=http-enum -p80,443 <target>
sudo nmap --script vuln <target>   # runs scripts in the vuln category (can be noisy/slow)
```

- Combine version detection and scripts:

```bash
sudo nmap -sV --script=banner,http-title,http-enum -p80,443 <target>
```

## Useful flags

- `--open` — show only open (or possibly open) ports.
- `--reason` — include why Nmap thinks a port is in that state.
- `-v` / `-vv` — increase verbosity.
- `-d` — debugging output for troubleshooting scan behavior.

## Host discovery options

- ICMP echo ping: `-PE` (default for ping scanning)
- TCP SYN to port 443: `-PS443` (useful if ICMP is blocked)
- TCP ACK: `-PA80,443`

Example: combine probes and skip discovery fallback:

```bash
sudo nmap -Pn -PS80,443 -PA3389 --max-retries 2 <target>
```

## Evasion and spoofing (use only when authorized)

- Fragment packets: `-f`
- Fake source address (IP spoofing): `--source-ip x.x.x.x` (rarely useful and often filtered)
- Use decoys: `-D RND:10` (generate 10 random decoys)
- Spoof MAC address: `--spoof-mac 0` or a vendor hex

Note: many evasion techniques can trigger alerts and are not reliable on modern networks.

## Examples / Recipes

- Quick but informative scan (default scripts, version detection, top ports):

```bash
sudo nmap -sC -sV --top-ports 200 -oA nmap/quick <target>
```

- Full TCP port/service sweep and save output:

```bash
sudo nmap -p- -sS -sV -T4 -oA nmap/full-tcp <target>
```

- UDP + TCP combined (long):

```bash
sudo nmap -sU -sS -p- -T3 -oA nmap/tcp-udp <target>
```

- Run vulnerability scripts against discovered services (careful; can be intrusive):

```bash
sudo nmap -sV --script=vuln -oA nmap/vuln <target>
```

## Integrations

- XML output (`-oX`) can be imported into Metasploit, Nessus, and other tools.
- Use `ndiff` to compare scans over time.

## Quick tips

- Always respect scope and authorization. Aggressive or intrusive scans can cause outages.
- Start conservative (top ports, low T) and increase intensity when safe.
- Save raw outputs and include command lines in your notes for reproducibility.

---
Guide: practical Nmap commands and guidance for enumeration tasks.
