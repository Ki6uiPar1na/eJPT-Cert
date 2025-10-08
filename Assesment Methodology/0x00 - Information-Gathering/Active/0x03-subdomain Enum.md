# 0x03 — Subdomain & VHost Enumeration

Goal: enumerate subdomains and virtual hosts to expand the attack surface.

Passive subdomain sources
- Certificate transparency (crt.sh)
- DNS historical services (securitytrails)
- Passive tools: `subfinder`, `amass -passive`

Active/brute-force techniques
- gobuster (DNS mode):
  - `gobuster dns -d example.com -w /path/to/wordlist -t 50 -o gobuster-dns.txt`
- gobuster vhost (vhost mode):
  - `gobuster vhost -u https://example.com -w /path/to/wordlist --exclude-length 1542 -o gobuster-vhost.txt`
- ffuf / dnsx / massdns for high-performance enumeration

Recommended workflow
1. Run passive discovery (crt.sh, amass -passive, subfinder).
2. Normalize and deduplicate findings.
3. Run targeted bruteforce with a curated wordlist (e.g., SecLists' DNS/raft-large) and reasonable threading.
4. Validate each candidate by resolving DNS and checking HTTP response fingerprints.

Validation examples
- Resolve and check HTTP status:
  - `host sub.example.com && curl -sI https://sub.example.com | head -n 5`

Notes
- Avoid aggressive threads on public infrastructure; respect rate limits and scope.
- Virtual host enumeration (vhost) can reveal forgotten apps hosted on the same IP.