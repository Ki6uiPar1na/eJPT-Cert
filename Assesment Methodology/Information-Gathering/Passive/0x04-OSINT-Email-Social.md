# 0x04 OSINT: Email Harvesting & Social Media

Purpose: gather email addresses, social profiles, and leaked credentials without interacting with target services.

Tools & commands:

- theHarvester
  - Install: `sudo apt install theharvester` or `pip3 install theharvester`
  - Usage: `theharvester -d example.com -l 200 -b google`
  - Example: `theharvester -d target.example -b all` searches multiple data sources (Google, Bing, LinkedIn, etc.).

- Have I Been Pwned (HIBP)
  - Website: `https://haveibeenpwned.com`
  - Use to check if an email was in a known breach. For automation, use API with key.

- Search engines and social media
  - Google dorks (see separate file) for finding email addresses and profiles.

Notes:
- Respect privacy and legal limits; harvesting public emails is allowed, but using them for malicious actions is illegal.
