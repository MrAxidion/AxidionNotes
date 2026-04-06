# CTF Writeup : Eloquia 

> **Author:** Mr Axidion  
> **Date:** 2025-12-18  
> **Difficulty:** Insane  

---

## 1. Initial Enumeration

The assessment started with an `nmap` scan to identify open ports and services:

- **Port 80 (HTTP):** Microsoft IIS 10.0 (main website)  
- **Port 5985 (WinRM):** Windows Remote Management  

After adding the following domains to `/etc/hosts`:

```
eloquia.htb
qooqle.htb
```

Two separate web applications were discovered.

---

## 2. Foothold: OAuth Exploitation

The application `eloquia.htb` supports authentication via `qooqle.htb`.

During analysis of the OAuth flow, a vulnerability was identified that allowed bypassing authentication.

- A custom Python script was used to generate a valid `access_token`  
- This allowed authentication as **Administrator** in the web panel  

---

## 3. Remote Code Execution via SQL Explorer

Inside the admin panel, a feature named **SQL Explorer** was discovered, allowing execution of SQLite queries.

### Exploitation Strategy

- A malicious `shell.dll` was created using C and compiled with `mingw`  
- The DLL was uploaded through the image upload functionality  

### Execution

The SQLite `load_extension` feature was abused:

```sql
SELECT load_extension('C:\\Web\\Eloquia\\static\\assets\\images\\blog\\shell.dll');
```

### Result

A reverse shell was obtained as the `web` user.

---

## 4. Lateral Movement (User Access)

Credentials were extracted from Microsoft Edge browser files:

- `Local State`  
- `Login Data`  

Using a decryption script (DPAPI), valid credentials were recovered.

Access was gained via WinRM:

```bash
evil-winrm -i <TARGET_IP> -u Olivia.KAT -p <PASSWORD>
```

**User Flag:**

```
C:\Users\Olivia.KAT\Desktop\user.txt
```

---

## 5. Privilege Escalation

### Vulnerability Discovery

During permission analysis, it was found that the user `Olivia.KAT` had **write access** to the following binary:

```
C:\Program Files\Qooqle IPS Software\Failure2Ban - Prototype\Failure2Ban\bin\Debug\Failure2Ban.exe
```

This binary is executed by a service running with SYSTEM privileges.

---

### Exploitation: Service Binary Hijacking

#### Step 1: Malicious Binary

A custom C program was created to copy the root flag:

- Source: `C:\Users\Administrator\Desktop\root.txt`  
- Destination: `C:\temp\root.txt`  

---

#### Step 2: Race Condition

Since the service binary was in use, a PowerShell loop was used to overwrite it when possible:

```powershell
while($true){ try { Copy-Item $s $t -Force -ErrorAction Stop; break } catch { Start-Sleep -Milliseconds 200 } }
```

---

#### Step 3: Execution

Once the service restarted, it executed the malicious binary with **SYSTEM privileges**, resulting in the flag being copied.

---

## 6. Root Access

**Root Flag:**

```
C:\temp\root.txt
```

---

## Conclusion

This machine required a combination of skills across multiple domains:

- Web exploitation (OAuth bypass)  
- Database abuse (SQLite extensions)  
- Credential extraction (DPAPI)  
- Windows privilege escalation (service binary hijacking)  

It highlights the risks of insecure authentication flows, unsafe database features, and improper file permissions on privileged services.
