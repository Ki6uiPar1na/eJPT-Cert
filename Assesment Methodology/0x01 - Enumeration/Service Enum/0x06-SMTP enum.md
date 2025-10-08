# 0x06 - SMTP enumeration

Below I preserved your original notes (above) exactly as you wrote them. The section below is a merged, cleaned, and expanded version that organizes steps, commands, and native exploitation approaches for lab and engagement use.

## What is SMTP?

SMTP (Simple Mail Transfer Protocol) is the protocol used to send email between mail servers. Common server software includes Postfix, Exim, Sendmail, and Microsoft Exchange.

Why enumerate SMTP?

- Fingerprinting mail server software and versions.
- Enumerating valid mailboxes/users (VRFY/EXPN or RCPT-based probing).
- Finding misconfigurations (open relay, weak TLS, permissive auth).
- Obtaining/validating credentials that can be used for sending mail or accessing IMAP/POP3.

Default ports

- 25 — SMTP (plain / STARTTLS)
- 465 — SMTPS (implicit TLS)
- 587 — Submission (STARTTLS, authenticated)

## High-level process

1. Discovery & versioning (nmap + NSE)
2. Banner and TLS inspection (openssl s_client)
3. Manual interaction (telnet/nc) to test VRFY/EXPN/RCPT
4. Automated enumeration (smtp-user-enum, nmap scripts, Metasploit)
5. Authentication testing (swaks, Metasploit smtp_login)
6. Abuse vectors (open relay, authenticated sending) and post-enum (IMAP/POP3)

## Commands and examples

1) Discovery / fingerprinting

```bash
# quick connectivity
nc -vz demo.ine.local 25

# version + NSE checks
nmap -sV -p 25,465,587 --script=banner,smtp-commands,smtp-enum demo.ine.local
```

Sample expected output (nmap NSE):

```
PORT    STATE SERVICE
25/tcp  open  smtp
| smtp-commands:
|   supported command: EHLO, MAIL, RCPT, DATA, VRFY, EXPN, STARTTLS
|_  STARTTLS supported
| smtp-enum:
|   Found 3 users: alice, bob, eve
```

2) Banner + STARTTLS inspection

```bash
openssl s_client -starttls smtp -crlf -connect demo.ine.local:25

# implicit TLS on 465
openssl s_client -connect demo.ine.local:465
```

Look for SANs/CN, weak ciphers, TLS version, and issuer information.

3) Manual protocol checks (VRFY / RCPT)

Use `nc` or `telnet` to talk directly to the MTA and observe responses:

```bash
nc demo.ine.local 25

# then type:
EHLO attacker.example
VRFY alice
RCPT TO:<alice@example.com>
```

Interpretation:

- 250 indicates success (likely valid user).
- 550 indicates unknown recipient.
- Some servers disable VRFY/EXPN; use `RCPT TO` probing instead.

4) Automated enumeration

smtp-user-enum (classic):

```bash
smtp-user-enum -M RCPT -U users.txt -t demo.ine.local
smtp-user-enum -M VRFY -U users.txt -t demo.ine.local
```

nmap script (smtp-enum):

```bash
nmap -p 25 --script=smtp-enum --script-args='smtp-enum.usersdb=/usr/share/wordlists/smtp-users.txt' demo.ine.local
```

swaks (diagnostics & auth checks):

```bash
swaks --server demo.ine.local:25 --from test@attacker.com --to victim@example.com

# auth test
swaks --server demo.ine.local:25 --auth LOGIN --auth-user alice --auth-password 'P@ssw0rd'
```

5) Metasploit quick flow

```bash
msfconsole -q
workspace -a smtp-demo
setg RHOSTS demo.ine.local

# version
use auxiliary/scanner/smtp/smtp_version
run

# enumeration
use auxiliary/scanner/smtp/smtp_enum
set USER_FILE /path/to/users.txt
set THREADS 10
run

# auth brute (careful)
use auxiliary/scanner/smtp/smtp_login
set USER_FILE /path/to/users.txt
set PASS_FILE /path/to/passwords.txt
run
```

Notes: `smtp_enum` can use VRFY/EXPN/RCPT techniques; `smtp_login` will attempt AUTH mechanisms.

6) Native exploitation / abuse examples

- RCPT-based mailbox probing (bash loop):

```bash
while read user; do
  printf "EHLO test.example\r\nMAIL FROM:<me@test.example>\r\nRCPT TO:<${user}@example.com>\r\nQUIT\r\n" | nc demo.ine.local 25 | grep -q "250" && echo "FOUND: ${user}" || echo "NOTFOUND: ${user}"
done < users.txt
```

- Open-relay test (swaks or netcat):

```bash
swaks --from test@attacker.com --to victim@external.com --server demo.ine.local:25

printf "HELO attacker\r\nMAIL FROM:<test@attacker.com>\r\nRCPT TO:<victim@external.com>\r\nDATA\r\nSubject: test\r\n\r\nHello\r\n.\r\nQUIT\r\n" | nc demo.ine.local 25
```

- Authenticated sending (Python example):

```python
import smtplib

server = 'demo.ine.local'
port = 587
user = 'alice'
passwd = 'P@ssw0rd'

s = smtplib.SMTP(server, port)
s.ehlo()
s.starttls()
s.login(user, passwd)
s.sendmail('attacker@example.com', 'victim@example.com', 'Subject: test\n\nThis is a test')
s.quit()
```

Note: SMTP itself is not for mailbox reads — use IMAP/POP3 if you need to inspect messages.

## Post-enumeration and pivot ideas

- If credentials are valid: try IMAP/POP3 to read mailboxes, use as phishing relay (with authorization), or extract tokens and PII.
- If open relay: document exploitation steps and capture an example message for reporting.

## Defenses & detection

- Disable/limit VRFY/EXPN; rate-limit RCPT checks.
- Enforce STARTTLS and strong ciphers.
- Require authentication for relaying; monitor outbound mail patterns.

## Troubleshooting & tips

- Use RCPT-based checks when VRFY is disabled.
- Keep thread counts low to avoid lockouts or IDS triggering.
- Test authenticated flows only with explicit permission.

## Quick cheat-sheet

```bash
nmap -sV -p 25,465,587 --script=smtp-commands,smtp-enum demo.ine.local
openssl s_client -starttls smtp -connect demo.ine.local:25
nc demo.ine.local 25
smtp-user-enum -M RCPT -U users.txt -t demo.ine.local
swaks --server demo.ine.local:25 --auth LOGIN --auth-user alice --auth-password 'P@ss'
```
