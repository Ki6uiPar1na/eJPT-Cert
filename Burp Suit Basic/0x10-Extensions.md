# Extensions

Purpose

Burp extensions (BApps or manual plugins) expand Burp Suite’s functionality. Extensions can add new scanners, automation helpers, or integrate external tools.

Installing extensions

- From BApp Store:

  1. Open Extensions → BApp Store.
  2. Browse or search for an extension and click Install.

- Manual installation:

  1. Download the extension (.jar for Java, .py for Python/Jython).
  2. Open Extensions → Add.
  3. Choose the extension type, point to the file, and install.

Popular extensions and use cases

- Turbo Intruder — high-performance request sender for large-scale automated tests (advanced workflows).
- SQLMap for Burp — integrates SQLMap features to detect and exploit SQL injection from within Burp.
- Autorize — automates authorization checks to detect access control issues.
- Active Scan++ — broadens Burp’s active scanning capabilities with more checks.
- 403 Bypasser — tools and test strategies to try bypassing HTTP 403 restrictions.

Managing extensions

- Update: BApp store extensions can be updated via the Extensions tab; manual extensions must be reinstalled when new Versions are available.
- Remove: Select an extension in the list and click Remove.
- Usage: Many extensions add a new tab or UI element—read the extension’s documentation for details.

Security note

- Evaluate third-party extensions before installing; verify source and compatibility. Malicious extensions could compromise your testing environment.

Tip: Extensions are powerful—use them to cover gaps in Burp's default capabilities, but keep them updated and audited.



