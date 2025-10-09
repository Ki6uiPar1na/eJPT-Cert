# Directory Scanning — Cheat Sheet

## Table of contents
1. [Introduction](#introduction)
2. [Key concepts](#key-concepts)
   - [Directory scanning](#directory-scanning)
   - [Content discovery](#content-discovery)
   - [Fuzzing & recursion](#fuzzing--recursion)
3. [Wordlists & sizing](#wordlists--sizing)
4. [Quick manual checks](#quick-manual-checks)
   - [robots.txt](#robotstxt)
   - [sitemap.xml](#sitemapxml)
5. [Tools & concise examples](#tools--concise-examples)
   - [Gobuster](#gobuster)
   - [FFUF](#ffuf)
   - [Feroxbuster](#feroxbuster)
   - [Dirb](#dirb)
6. [File / extension scanning examples](#file--extension-scanning-examples)
7. [Workflow & practical tips](#workflow--practical-tips)
8. [Quick cheat-sheet commands](#quick-cheat-sheet-commands)
9. [References & wordlists](#references--wordlists)

---

## Introduction

Compact, practical guidance for directory and content discovery during web application assessments. Use only on systems you are authorized to test.

## Key concepts

### Directory scanning

Probe likely paths (e.g., /admin, /backup, /uploads) to find hidden pages or sensitive files. Treat discoveries as leads and verify manually.

### Content discovery

Combine sitemap crawling, robots.txt review, JS parsing, and link aggregation to build high-quality wordlists.

### Fuzzing & recursion

Fuzzing substitutes a token (e.g., FUZZ) with entries from a wordlist. Recursion follows discovered directories — use selectively to limit noise.

## Wordlists & sizing
- Start small (common.txt, raft-small) for quick wins; escalate to directory-list-1.0/2.3 for deeper coverage.
- Keep SecLists locally: `/root/Desktop/misc/SecLists` or `/usr/share/wordlists/SecLists`.
- Stage large lists: `split -l 50000 biglist chunk-` or sample with `shuf -n 10000 biglist`.

## Quick manual checks

### robots.txt

```bash
curl -sS https://target.com/robots.txt | sed -n '1,120p'
```

### sitemap.xml

```bash
curl -sS https://target.com/sitemap.xml | xmllint --format - 2>/dev/null | sed -n '1,120p'
```

## Tools & concise examples

Replace `target.com`, `http://IP`, and `/path/to/wordlist` before running.

### Gobuster — fast brute-forcer (dir/dns/vhost/s3)

Install: `apt install gobuster` or `go install github.com/OJ/gobuster/v3@latest`

```bash
gobuster dir -u http://target.com -w /path/to/common.txt -t 40 -o gobuster-dir.txt
gobuster dir -u http://target.com -w /path/to/common.txt -x php,html -t 40 -o gobuster-files.txt
gobuster dns -d target.com -w /path/to/subdomains.txt -t 50 -o gobuster-dns.txt
```

Notes: very fast; no built-in recursion — script repeated runs for recursion.

### FFUF — flexible, high-performance fuzzing

Install: `go install github.com/ffuf/ffuf@latest`

```bash
ffuf -u http://target.com/FUZZ -w /path/to/directory-list-1.0.txt -t 60 -o ffuf-dir.json -of json
ffuf -u http://target.com/indexFUZZ -w /path/to/web-extensions.txt -t 40 -o ffuf-ext.json
ffuf -u http://target.com/FUZZ -w /path/to/common.txt -mc 200,301,302 -fc 404 -t 80
```

Notes: use `-mc`/`-fc`/`-ml` to reduce noise; JSON output is easy to parse.

### Feroxbuster — recursive, robust discovery

Install: prebuilt binary or `cargo install feroxbuster`

```bash
feroxbuster -u http://target.com -w /path/to/common.txt -t 50 -r -o ferox.json
feroxbuster -u http://target.com -w /path/to/common.txt -t 30 -o ferox-quick.txt
```

Notes: respects robots.txt by default; recursion adds depth and time/noise.

### Dirb — simple baseline scanner

Install: `apt install dirb`

```bash
dirb http://target.com /path/to/common.txt -o dirb.txt
dirb http://target.com /path/to/common.txt -X .html -o dirb-html.txt
```

Notes: quick baseline checks; good for legacy environments.

## File / extension scanning examples

```bash
gobuster dir -u http://172.20.8.56 -w /root/Desktop/misc/SecLists/Discovery/Web-Content/common.txt -x php -t 30 -o gobuster-php.txt
ffuf -u http://172.20.3.144/indexFUZZ -w /root/Desktop/misc/SecLists/Discovery/Web-Content/web-extensions.txt -t 40 -o ffuf-index.json
ffuf -u http://172.20.3.144/FUZZ.html -w /root/Desktop/misc/SecLists/Discovery/Web-Content/common.txt -t 40 -v
feroxbuster -u http://172.20.3.144 -w /root/Desktop/misc/SecLists/Discovery/Web-Content/common.txt -t 40 -r -o ferox-recursive.txt
```

## Workflow & practical tips

1. Manual checks: robots.txt, sitemap.xml, visible JS and public endpoints.
2. Start with small lists and low threads for quick feedback.
3. Triage and verify results manually; run targeted extension/parameter scans.
4. Use recursion sparingly — deep scans are noisy and time-consuming.
5. Store outputs (`-o` / JSON) for parsing and reporting.

## Quick commands

```bash
gobuster dir -u http://target.com -w /path/to/common.txt -t 30 -o gobuster-quick.txt
ffuf -u http://target.com/FUZZ -w /path/to/common.txt -mc 200,301,302 -t 60 -o ffuf-quick.json
feroxbuster -u http://target.com -w /path/to/wordlist.txt -t 50 -r -o ferox-deep.txt
```

## References & wordlists
- SecLists: https://github.com/danielmiessler/SecLists (Discovery/Web-Content)
- Small lists: `common.txt`, `raft-small-words.txt`

---


