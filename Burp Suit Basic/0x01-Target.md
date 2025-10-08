# Target

Purpose

The Target tool helps you build a structural map of the web application and define the scope of testing. Proper use of Target focuses testing and reduces accidental interaction with out-of-scope systems.

Main areas

- Site Map
  - A hierarchical tree of discovered URLs, directories, and files.
  - Click any node to view request/response details.
  - Right-click a resource to send it to Repeater, Intruder, Decoder, Comparer, or other tools.
  - Use filters to hide static assets (images, fonts, CSS) and show only relevant request types (GET/POST).

- Scope
  - Explicitly add hosts/URL patterns that are in-scope for automated or manual testing.
  - Use advanced filters for fine-grained control (specific paths, query parameters).
  - Many Burp features respect the defined scope—set it before significant automated actions.

Issue Definitions

- Burp includes a knowledge base of issue types with descriptions, references, and severity ratings. Use these when triaging findings and preparing reports.

Practical tips

- Keep the Site Map readable by excluding third-party domains and static assets.
- Add only authorized hosts to Scope; accidental scans of external services are a common legal risk.
- Use Site Map as your canonical view for what you’ve explored and confirmed during an engagement.



