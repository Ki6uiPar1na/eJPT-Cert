# Sequencer

Purpose

Sequencer evaluates the randomness of tokens (session IDs, CSRF tokens, password reset tokens) by running statistical tests against a large sample of values.

Workflow

1. Capture token-bearing responses via Proxy (or other tools).
2. Right-click the token → Send to Sequencer.
3. In Sequencer, collect a large sample set (thousands of tokens are recommended for meaningful statistics).
4. Run the analysis — Burp applies multiple statistical tests and generates a report on entropy and predictability.

Interpreting results

- Poor randomness / low entropy suggests tokens may be predictable and vulnerable to guessing or replay.
- Sequencer provides tests that help determine whether the token source is cryptographically secure or not.

Practical tips

- Ensure tokens are captured under realistic usage patterns (e.g., new session creation, login, or token refresh flows).
- Collect a sufficiently large sample for statistical validity; small samples often yield misleading results.

