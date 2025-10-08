# Decoder

Purpose

Decoder provides quick encode/decode/transformation utilities for payloads you encounter while testing. It is helpful for unraveling obfuscated data and preparing payloads for injection.

Input methods

- Manual entry: paste or type text directly into the Decoder tab.
- Send from other tools: right-click a value in Proxy/Repeater and Send to Decoder.

Primary features

- Smart Decode: attempts to auto-detect and decode common encodings.
- Manual transforms available:
  - URL Decode / Encode
  - HTML Decode / Encode
  - Base64 Encode / Decode
  - ASCII Hex, Hex, Binary conversions
  - GZIP decompress
- Hashing functions: MD2, MD4, MD5, SHA-1, SHA-256, SHA-512 (useful to compare hashed values or verify known hashes).

Examples

- Copy a Base64 cookie value into Decoder → Base64 Decode → inspect JSON/payload.
- Take an encoded URL parameter → URL Decode → check for hidden JSON or SQL fragments.

Tip: Use Decoder before automating payloads—ensure the payload is in the server-expected encoding.
