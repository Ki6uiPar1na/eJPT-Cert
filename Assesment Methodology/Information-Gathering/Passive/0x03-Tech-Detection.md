# 0x03 Web Technology Detection (WhatWeb, Wappalyzer)

Purpose: identify frameworks, libraries, server software, and other technologies used by a website.

Tools & commands:

- WhatWeb (CLI)
  - Install: `sudo apt install whatweb` or `gem install whatweb`
  - Usage: `whatweb https://example.com`
  - Example output: shows CMS (WordPress), server (nginx), JS libraries, analytics, plugins.

- Wappalyzer (browser extension)
  - Install in Firefox/Chrome and visit the site to see detected technologies in the toolbar.

- BuiltWith and online services offer similar info via web UI.

Tips:
- Combine WhatWeb with manual inspection of HTML (view-source) for meta tags, script URLs, and comments.
