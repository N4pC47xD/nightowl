# 🦉 NightOwl
> Post-Exploitation Enumerator for CTF / Penetration Testing

**Author:** N4pC47xD  
**Version:** 1.0  
**Platform:** Linux / Windows  

> ⚠️ For authorized use only. Only run on systems you own or have explicit written permission to test.

---

## What is NightOwl?

NightOwl is a single-file, zero-dependency Python post-exploitation enumeration tool designed for HTB, CTF, and authorized penetration testing engagements. Drop it on a box and it tells you everything worth knowing — from privilege escalation paths to internal services, container pivot targets, and vulnerable code patterns.

---

## Features

| Module | Flag | What it finds |
|---|---|---|
| System Info | `--system` | OS, kernel, distro, CPU, uptime, mounts, container detection |
| User Enumeration | `--users` | Login shells, home dirs, .ssh / .aws / .bash_history presence, active logins |
| Group Enumeration | `--groups` | Current groups flagged against interesting list (docker, lxd, sudo...) |
| Sudo / PrivEsc | `--sudo` | sudo -l, SUID/SGID binaries, file capabilities, writable sensitive files |
| Interesting Binaries | `--binaries` | GTFOBins-style binary check, writable PATH dirs, non-standard executables |
| Scheduled Jobs | `--jobs` | System/user cron, systemd timers, at queue |
| Writable Paths | `--writable` | World-writable dirs, writable files in /etc /bin /usr, NFS no_root_squash |
| Services & Ports | `--services` | Localhost-only ports (invisible to nmap), root processes, systemd units |
| Credential Hunting | `--creds` | .env, wp-config, history files, /etc/shadow, AWS keys, config file passwords |
| Network Info | `--network` | Interfaces, routes, ARP cache, /etc/hosts, SSH known_hosts |
| Installed Software | `--software` | Package list, key binary versions |
| Environment | `--env` | Sensitive env vars, LD_PRELOAD hijack check, shell aliases |
| Container Discovery | `--container` | Docker socket, sibling containers, internal subnets, LXD group check |
| Config & Code Analysis | `--configs` | Scans code/config files for SQLi, RCE, LFI, SSTI, XXE, SSRF, hardcoded creds |

---

## Requirements

- Python 3.x (no external libraries required)
- Linux or Windows target

---

## Installation

```bash
# Download the tool
git clone https://github.com/N4pC47xD/nightowl.git
cd nightowl

# Make executable
chmod +x nightowl

# Optional: add to PATH for global access
sudo cp nightowl /usr/local/bin/nightowl
```

---

## Usage

```bash
# Show help
nightowl --help

# Run all modules
nightowl --all

# Run specific modules
nightowl --users --sudo --services

# Save output to timestamped file (screen + clean text file)
nightowl --all --output

# No color (for manual piping)
nightowl --all --no-color | tee output.txt
```

---

## Output Key

```
[*]  Informational
[+]  Notable finding — worth investigating
[!]  High-value / likely PrivEsc path
```

---

## Output Example

```
    ,___,
    [O.O]   N I G H T O W L
    /)  (\  ─────────────────────────────────
  -"--  --"-  Post-Exploitation Enumerator
               version 1.0  //  author N4pC47xD
               authorized systems only

════════════════════════════════════════════════════════════
  SUDO / PRIVILEGE ESCALATION PATHS
════════════════════════════════════════════════════════════

  [!] SUDO RULE:  (ALL) NOPASSWD: /usr/bin/vim
  [+] Non-standard SUID: /usr/bin/python3.8
  [+] Capability: /usr/bin/python3 cap_setuid+ep
```

---

## --configs Module

The `--configs` module walks common application directories and scans code and configuration files for dangerous patterns. Each finding shows the file, line number, severity, and surrounding context.

**Scanned locations:** `/var/www`, `/opt`, `/srv`, `/app`, `/api`, `/backend`, `/home`, `/etc`, `/usr/local`

**Detected patterns:**

- **CRITICAL** — RCE (`exec`, `system`, `shell_exec`, `os.system`, `subprocess shell=True`, `eval`), SQL injection, LFI/RFI (`include`/`require` with user input), deserialization (`pickle.loads`, `unserialize`), SSTI (`render_template_string`), SSRF
- **WARNING** — Unsafe YAML load, file path concatenation, XML entity resolution, hardcoded credentials
- **INFO** — Weak crypto (MD5/SHA1), non-crypto random

```
  ┌─ /var/www/html/search.php
  │  [CRITICAL] line 12 — system() with variable input
  │       10      $term = $_GET['q'];
  │       11      // run search
  │  12 ▶         system("grep -r " . $term . " /var/data");
  │       13  }
  │
  └──────────────────────────────────────────────────────────
```

---

## --output Flag

Running with `--output` automatically saves a clean, color-free copy to a timestamped file in the current directory:

```
nightowl_<hostname>_<timestamp>.txt
```

Colors still render on screen — only the file is stripped of ANSI codes.

---

## Disclaimer

This tool is intended for use on systems you own or have been given explicit written authorization to test. Unauthorized use against systems you do not own is illegal. The author accepts no responsibility for misuse.
