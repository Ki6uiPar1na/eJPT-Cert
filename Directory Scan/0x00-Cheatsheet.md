# Directory Scan — Cheat Sheet
````markdown
# Directory Scan — Cheat Sheet

## Table of contents
1. [Introduction](#introduction)
2. [Key concepts](#key-concepts)
   - [Directory scanning](#directory-scanning)
   - [Content discovery](#content-discovery)
   - [Fuzzing](#fuzzing)
3. [Wordlists](#wordlists)
4. [Manual discovery checks](#manual-discovery-checks)
   - [robots.txt](#robotstxt)
   - [sitemap.xml](#sitemapxml)
5. [Tools & examples](#tools--examples)
   - [Gobuster](#gobuster)
   - [FFUF](#ffuf)
   - [Feroxbuster](#feroxbuster)
   - [Dirb](#dirb)
6. [Workflow & tips](#workflow--tips)
7. [Quick cheat-sheet commands](#quick-cheat-sheet-commands)
8. [References](#references)

---

## Introduction

Directory scanning (content discovery / web fuzzing) helps find hidden files, forgotten pages, and misconfigurations that can lead to security issues. It's often one of the earliest phases in a web application assessment and provides high-value leads for follow-up testing.

This cheat sheet consolidates definitions, recommended tools, useful commands, and practical tips for directory and content discovery.

## Key concepts

### Directory scanning
Probe common or guessed paths (e.g., /admin, /uploads, /.env) to discover files and directories that are not linked from the site. Files discovered this way may expose sensitive info or administration interfaces.

### Content discovery
A broader process that includes directory scanning, sitemap crawling, robots analysis, subdomain discovery, and crawling outputs to locate useful resources.

### Fuzzing
Sending many crafted inputs (candidate filenames, parameter values, headers) and observing responses to reveal hidden behavior or resources. Directory fuzzing is brute-forcing path names using wordlists.

## Wordlists

Use curated lists (SecLists) and tune by size and scope.
- Start small (common.txt) for quick wins; then escalate to bigger lists (directory-list-1.0/2.3) for deeper coverage.
- Typical path on lab images: `/usr/share/wordlists/` or your local SecLists copy.

Tip: start with smaller lists (common.txt) for speed, then increase list size for depth.

## Manual discovery checks

Before mass scanning, inspect these resources manually — they often contain direct clues.

### robots.txt

- Often lists `Disallow` entries that point to admin pages, staging, or forgotten resources.

```bash
curl -sS https://target.com/robots.txt
```

Example snippet:

```
User-agent: *
Disallow: /admin
Disallow: /private_data
Disallow: /temp
```

### sitemap.xml

- Sitemap provides canonical URLs and can reveal hidden pages not linked normally.

```bash
curl -sS https://target.com/sitemap.xml
```

## Tools & examples (concise)

Below are the most commonly used tools for directory/content discovery with concise example commands.

### Gobuster — fast brute-forcer (dir/dns/vhost/s3)
- Install hint: `apt install gobuster` (or `go install github.com/OJ/gobuster/v3@latest`)
- Quick commands:

```bash
# directory scan
gobuster dir -u http://target.com -w /path/to/wordlist.txt -t 50 -o gobuster-dir.txt

# with extensions
gobuster dir -u http://target.com -w /path/to/common.txt -x php,html -t 50 -o gobuster-files.txt

# DNS/subdomain mode
gobuster dns -d target.com -w /path/to/subdomains.txt -t 50
```

- Note: very fast, no built-in recursion — use scripted recursion or follow-up scans.

### FFUF — flexible HTTP fuzzing (FUZZ keyword)
- Install hint: `go install github.com/ffuf/ffuf@latest` or use packed binaries
- Quick commands:

```bash
# directory discovery
ffuf -u http://target.com/FUZZ -w /path/to/wordlist.txt -t 50 -o ffuf-dir.json -of json

# extension enumeration (indexFUZZ -> index.html|index.php)
ffuf -u http://target.com/indexFUZZ -w /path/to/web-extensions.txt -t 40

# filter common statuses (keep 200/301/302)
ffuf -u http://target.com/FUZZ -w /path/to/wordlist.txt -mc 200,301,302 -t 100
```

- Note: use `-mc` (match-codes), `-fc` (filter-codes), `-ml` (match-length) to reduce noise.

### Feroxbuster — recursive, robust scanning
- Install hint: prebuilt binaries available or `cargo install feroxbuster`
- Quick commands:

```bash
# basic scan
feroxbuster -u http://target.com -w /path/to/wordlist.txt -t 40 -o ferox.txt

# recursive deep scan
feroxbuster -u http://target.com -w /path/to/wordlist.txt -t 50 -r -o ferox-recursive.txt
```

- Note: excellent for deep discovery and recursive exploration; respects robots.txt by default unless overridden.

### Dirb — simple scanner with built-in lists
- Install hint: usually preinstalled on Kali/HackerBox or `apt install dirb`
- Quick commands:

```bash
# basic
dirb http://target.com /path/to/wordlist.txt -o dirb.txt

# look for specific extension
dirb http://target.com /path/to/wordlist.txt -X .html -o dirb-html.txt
```

- Note: good for quick checks and legacy environments; less flexible than ffuf/gobuster.

### Common flags (quick reference)

- `-u` URL / `-d` domain (gobuster)
- `-w` / `--wordlist` path to wordlist
- `-t` threads (concurrency)
- `-o` output file
- `-x` extensions (gobuster)
- `-r` recursive (feroxbuster)
- `-mc` / `-fc` match/filter status codes (ffuf)

Keep commands minimal and adapt thread counts to the target (lower on production).

## File / Page scanning (merged examples)

Use targeted scans when you want to look for specific file types or page names. Start with a conservative thread count and small lists, then scale up.

Gobuster: detect .php files (example)

```bash
gobuster dir -u http://172.20.8.56 -w /root/Desktop/misc/SecLists/Discovery/Web-Content/common.txt -x php -t 30 -o gobuster-php.txt
```

FFUF: directory discovery and extension scans

```bash
# directory discovery (FUZZ placeholder)
ffuf -u http://172.20.3.144/FUZZ -w /root/Desktop/misc/SecLists/Discovery/Web-Content/directory-list-1.0.txt -t 50 -mc 200,301,302 -o ffuf-dir.json -of json

# extension enumeration (indexFUZZ -> index.html, index.php...)
ffuf -u http://172.20.3.144/indexFUZZ -w /root/Desktop/misc/SecLists/Discovery/Web-Content/web-extensions.txt -t 40 -o ffuf-ext.json

# quick check for FUZZ.html variants (verbose)
ffuf -u http://172.20.3.144/FUZZ.html -w /root/Desktop/misc/SecLists/Discovery/Web-Content/common.txt -t 40 -v
```

Feroxbuster: recursive deep scan (concise)

```bash
feroxbuster -u http://target.com -w /path/to/wordlist.txt -t 40 -r -o ferox-recursive.txt
```

Dirb: simple baseline scans

```bash
dirb http://target.com /path/to/wordlist.txt -o dirb.txt
dirb http://target.com /path/to/wordlist.txt -X .html -o dirb-html.txt
```

## Notes and next steps

- Use small lists first, then run deeper scans on discovered directories only.
- Save output as JSON or text (`-o`) for later triage.
- Lower `-t` on production or unknown networks to avoid DoS/alerts.
- If you want, I can add `tools/` scripts that run safe starter scans (Gobuster/FFUF/Ferox) and store results in `loot/`.

---
````

ffuf -u http://172.20.3.144/FUZZ.html -w /root/Desktop/misc/SecLists/Discovery/Web-Content/common.txt -v

