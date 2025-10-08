# Repeater

Purpose

Repeater is the manual testing workhorse. Use it to edit a single request and resend it repeatedly while observing how the server responds.

Workflow

1. Capture or locate a request (Proxy or Site Map) and right-click → Send to Repeater.
2. In the Repeater tab, modify the request:
   - Change HTTP method (GET/POST/PUT), URL, headers, parameters, or body.
   - Edit cookies, authentication headers, or payloads.
3. Click Send to issue the request and view the response on the right.
4. Iterate: make small manual changes and re-send to observe differences.

Use cases

- Testing parameter tampering and input validation.
- Prototyping payloads for XSS/SQLi before automating them in Intruder.
- Verifying authorization checks by changing cookies or tokens.

Tips

- Keep a clear baseline request/response to compare changes.
- Use Repeater in combination with Comparer to highlight differences quickly.
- Repeater is ideal for slow, careful interaction—use Intruder when you need automation.

