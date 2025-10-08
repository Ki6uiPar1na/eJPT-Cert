# 0x03 — Web Technology Detection

Goal: fingerprint web technologies (CMS, frameworks, JS libs, analytics, server software) to plan targeted enumeration.

Tools
- WhatWeb (CLI)
  - Install: `sudo apt install whatweb` or `gem install whatweb`
  - Usage: `whatweb https://example.com`

- Wappalyzer (browser extension) — quick visual detection in browser

- curl for manual header / HTML inspection
  - `curl -sI https://example.com`  # headers
  - `curl -sL https://example.com | sed -n '1,200p'`  # first 200 lines of HTML

Indicators to inspect
- Response headers: `Server:`, `X-Powered-By:`, security headers
- HTML meta tags: `<meta name="generator" content="WordPress ...">`
- Script URLs and cookie names (some frameworks set distinct cookies)
- Assets & endpoints (e.g., `/wp-login.php`, `/administrator/`)

Example workflows
1. Run WhatWeb for an initial fingerprint:
   - `whatweb -v https://example.com`
2. Confirm using headers & HTML:
   - `curl -sI https://example.com`
   - `curl -sL https://example.com | grep -Ei "wp-|wordpress|joomla|django|rails|php" | head`

Common cookie indicators
- WordPress: cookies named `wordpress_*`, `wp-settings*`
- Django: cookie `csrftoken`
- Laravel: cookie `laravel_session`

Tips
- Combine automatic tools with manual inspection for accuracy.
- Keep an eye on CDN or WAFs that may mask server details. Try retrieving sub-resources (e.g., `/robots.txt`) for different hints.
