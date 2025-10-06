# 0x02 Website Footprinting (robots, sitemap, Netcraft)

Purpose: discover public site structure, disallowed paths, and hosting infrastructure.

Common checks:

- robots.txt
  - URL: `https://example.com/robots.txt`
  - What to look for: `Disallow:` entries can reveal hidden admin paths or directories.

- sitemap.xml
  - URL: `https://example.com/sitemap.xml`
  - What to look for: list of pages, language-specific paths, change frequency and priority.

- Netcraft
  - Use Netcraft extension or website to view server technologies, hosting provider, and historical records.

- Mirror / archive sites
  - Check Wayback Machine and Google cache (`cache:example.com`) to find past content.

- Download mirror (for offline analysis)
  - httrack example: `httrack "http://example.com" -O ./example_mirror`

Notes:
- Respect robots.txt for indexing but remember it doesn't prevent access — it only instructs crawlers.
