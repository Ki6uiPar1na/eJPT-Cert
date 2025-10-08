# Intruder

## Purpose

Intruder automates customized attack patterns against web requests. Typical uses include password guessing, parameter fuzzing, and automated payload injection. In the Community edition, Intruder is functional but slower; plan payload lists accordingly.

## Workflow

1. Send request to Intruder (right-click on a captured request → Send to Intruder).
2. Positions tab:
   - Burp will auto-suggest positions to mark with § markers.
   - Use Clear to remove automatic selections and add markers manually for precise control.
3. Choose an Attack Type:
   - Sniper — single position varied across payloads (good for single-parameter tests).
   - Battering Ram — same payload applied to multiple positions.
   - Pitchfork — multiple lists are iterated in parallel (lock-step).
   - Cluster Bomb — Cartesian product across multiple payload lists (exhaustive).
4. Payloads tab:
   - Load lists from local files (SecLists) or type values manually.
   - Payload processing: transformations (encodings) applied before sending.
5. Settings & Resource Pool:
   - Control threads and concurrency; use Resource Pool to avoid DoS and rate-limit your attack.
6. Start Attack and analyze results.

## Analyzing results

- Columns include status, length, and time — use these to spot anomalies.
- If all responses show the same status (e.g., 200), differences in response length, specific strings, or response time can indicate successful payloads.
- Use Comparer or inline response body checks (regex) to detect subtle successes.

## Practical example: login brute-force

- Position: password field only.
- Attack Type: Sniper.
- Payloads: common passwords list (e.g., SecLists 10k-most-common.txt).
- Inspect results for unique response sizes or success indicators in the body.

## Safety & ethics

- Limit concurrency and respect rate limits.
- Never run exhaustive high-rate attacks against production systems without explicit permission.



