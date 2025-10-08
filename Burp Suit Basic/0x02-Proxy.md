# Proxy

Purpose

The Proxy module intercepts, inspects, and modifies HTTP and HTTPS traffic between a client (browser or app) and the web server. It is the starting point for most interactive testing workflows.

Key concepts

- Intercept
  - Toggle intercept on/off to stop traffic and edit requests or responses manually.
  - Controls: Forward, Drop, Intercept On/Off.

- HTTP History
  - Chronological log of traffic that passed through the proxy.
  - Inspect details (request/response, headers, bodies) and right-click to send to other tools.

- WebSocket History
  - Inspect frames for applications that use WebSockets (chat, notifications, realtime features).

Proxy Options

- Proxy Listeners
  - Configure IP/port pairs for Burp to listen on (default 127.0.0.1:8080). Multiple listeners can be added.
- Request/Response interception rules
  - Define filters to ignore static files or capture only relevant traffic (by extension, method, status code, etc.).
- SSL/TLS and CA certificate
  - Export the Burp CA via Proxy → Options → Import / export CA certificate, or visit http://burp and download the certificate.
  - Install the certificate into your OS/browser certificate store for HTTPS interception.

Configuring your browser

- Manual proxy: set HTTP proxy to 127.0.0.1 and port to your Burp listener (commonly 8080).
- FoxyProxy: a browser extension that simplifies switching proxy profiles.
- Use the embedded Burp Browser when you want an environment pre-configured with the Burp CA and proxy settings.

Practical tips

- Use interception filters to avoid noisy captures (images, fonts, analytics).
- If HTTPS fails after installing the CA, ensure the cert is in the correct store (browser vs OS) and that the browser has been restarted.
- Use Proxy Options to tune listener addresses if testing on a remote VM or from multiple machines.

