# Comparer

Purpose

Comparer highlights differences between two pieces of data. It helps identify subtle changes in responses or payloads that indicate successful injections or logic differences.

Modes

- Bytes: Compare binary data at the byte level (images, files, encoded blobs).
- Words: Compare text-based differences (HTML, JSON), with color-highlighted diffs for easy inspection.

Usage

- Paste two values directly or send responses from Proxy/Repeater to Comparer.
- Select the comparison mode depending on the data type.
- Review highlighted differences to determine how the server behavior changed after an input modification.

Practical examples

- Compare a normal response to a response after injecting payloads to see specific alterations.
- Use Bytes mode to detect small binary differences after tampering with encoded elements.

Tip: When Intruder results are ambiguous, send the two candidate responses to Comparer to locate the minimal difference that indicates success.

