# 0x04 — OSINT: Email Harvesting & Social Media

Goal: collect email addresses, public profiles, and breach data to build a map of people and contacts relevant to the target.

Tools & commands
- theHarvester
  - Install: `sudo apt install theharvester` or `pip3 install theharvester`
  - Example: `theharvester -d example.com -l 200 -b all`

- Have I Been Pwned (HIBP)
  - Web: https://haveibeenpwned.com — check if an email was leaked
  - API: requires API key for automated queries

- LinkedIn / Twitter / GitHub search
  - Manual searches and company pages can reveal employee emails and roles

- Google Dorks (see the dorking file) — find exposed documents containing emails

Ethics & privacy
- Collecting public emails for security testing is acceptable, but using them for unauthorized access or spam is illegal.
- Always document the source and timestamp of any OSINT findings.

Quick workflow
1. Run `theharvester` for domain-level harvesting.
2. Search HIBP for found emails.
3. Correlate profiles in LinkedIn/GitHub/Twitter to identify targets for social engineering tests (with permission).
