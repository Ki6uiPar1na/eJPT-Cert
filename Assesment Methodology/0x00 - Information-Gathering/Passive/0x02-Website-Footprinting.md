# 0x02 — Website Footprinting

Goal: discover public structure, hidden paths, and historical content that reveal attack surface.

Common meta files
- robots.txt — may list directories bots should avoid (useful to discover admin paths)
  - `curl --silent --get https://example.com/robots.txt`
- sitemap.xml — enumerates site URLs and priorities
  - `curl --silent https://example.com/sitemap.xml`
- security.txt (RFC 9116) — responsible disclosure contacts
  - `curl --silent https://example.com/.well-known/security.txt`
- humans.txt — occasionally contains team or contact info
  - `curl --silent https://example.com/humans.txt`

Historical content
- Wayback Machine (https://web.archive.org) — useful to find removed pages, older endpoints, or forgotten panels.
- Google cache: `cache:example.com` (in Google search)

Crawling & mirroring (for offline analysis)
- httrack (mirror site):
  - `httrack "https://example.com" -O ./example_mirror --robots=0`
  - Note: `--robots=0` ignores robots.txt — only use for authorized engagements.

Header inspection
- Obtain server headers (may reveal server, frameworks, CDN):
  - `curl -sI https://example.com`

Search engine queries
- Use site: and inurl: operators to find specific file types or paths:
  - `site:example.com inurl:admin`

Notes & caution
- robots.txt is a hint only — it does not prevent access. Use entries as leads, not excuses to breach.
- Respect site terms and legal boundaries when mirroring or crawling.
