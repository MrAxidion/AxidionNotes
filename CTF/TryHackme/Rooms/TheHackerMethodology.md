# The Hacker Methodology  

> Axidion | 06-04-26  

[Room Link](https://tryhackme.com/room/hackermethodology)  

In this room, we learn about the process that hackers and penetration testers follow when targeting a system.

---

## Task 1: Methodology Overview  

The general process followed by penetration testers consists of:  

1. Reconnaissance  
2. Enumeration / Scanning  
3. Gaining Access (Exploitation)  
4. Privilege Escalation  
5. Covering Tracks  
6. Reporting  

**Question:** What is the first phase of the Hacker Methodology?  

```
Reconnaissance
```

---

## Task 2: Reconnaissance Overview  

Reconnaissance is all about gathering information using publicly available sources without directly interacting with the target.

Some common techniques and tools:  

- **Google Dorking**: Using advanced search queries to find sensitive data (PDFs, login pages, etc.)  
- **Wikipedia**: Understanding the company’s history  
- **Twitter / YouTube**: Checking latest updates and announcements  
- **LinkedIn**: Discovering employees, roles, and company structure  

Other useful tools:  

- PeopleFinder  
- who.is  
- sublist3r  
- hunter.io  
- builtwith.com  
- wappalyzer  

Reconnaissance usually relies on open-source intelligence (OSINT).

**Question:** Who is the CEO of SpaceX?  

```
Elon Musk
```

**Question:** What does sublist3r list?  

```
Subdomains
```

**Question:** What is it called when using Google to find vulnerabilities or specific data?  

```
Google Dorking
```

---

## Task 3: Enumeration and Scanning Overview  

At this stage, the attacker starts interacting with the target system to identify vulnerabilities.

This phase includes:  
- Scanning for open ports  
- Identifying services and versions  
- Enumerating users, shares, directories, etc.  

Common tools used:  

- **nmap**: Scanning ports, services, and versions (can also run vulnerability scripts)  
- **dirb**: Directory brute-forcing to discover hidden pages  
- **Metasploit**: Framework for scanning, exploitation, and post-exploitation  
- **ExploitDB**: Searching for public exploits  
- **Burp Suite**: Intercepting and modifying HTTP requests for web testing  
- **enum4linux**: Enumerating information from Windows/Samba systems  

**Question:** What does enumeration help determine about the target?  

```
Attack surface
```

**Question:** What company developed Metasploit?  

```
Rapid7
```

**Question:** What company developed Burp Suite?  

```
PortSwigger
```

---

## Task 4: Exploitation  

This is the phase where the attacker tries to gain access to the target system.

Even though it's often seen as the most “interesting” phase, it highly depends on the previous steps. Poor reconnaissance or enumeration will lead to failed exploitation.

Common tools:  

- **Metasploit**: One of the most widely used exploitation frameworks  
- **sqlmap**: Used to exploit SQL Injection vulnerabilities  

**Question:** What is one of the primary exploitation tools used by pentesters?  

```
Metasploit
```

---

## Task 5: Privilege Escalation  

After gaining initial access, the next goal is to escalate privileges.

Target accounts:  

- **Windows**: Administrator or SYSTEM  
- **Linux**: root  

Common techniques:  

- Cracking password hashes  
- Exploiting vulnerable services  
- Password spraying or credential reuse  
- Using default credentials  
- Finding SSH keys or secret credentials  
- Enumerating system configurations  

Examples of useful commands:  

- `ifconfig` → Network information  
- `find / -perm -4000 -type f 2>/dev/null` → Finding SUID binaries  

**Question:** In Windows, what is the other target account besides Administrator?  

```
SYSTEM
```

**Question:** What SSH-related item can allow login without a password?  

```
Keys
```

---

## Task 6: Covering Tracks  

In real-world ethical hacking, this phase is usually not performed.

Penetration testers work under strict rules of engagement and must:  

- Have explicit permission  
- Respect the defined scope  
- Stop once objectives are achieved  

Instead of covering tracks, the tester should:  

- Document all actions  
- Help clean up any artifacts left behind  
- Assist in remediation  

```
No answer needed
```

---

## Task 7: Reporting  

This is the final and one of the most important phases.

The report should clearly explain everything discovered during the test.

A good report includes:  

- Findings (vulnerabilities)  
- Severity (criticality)  
- Description of each issue  
- Steps to reproduce  
- Remediation recommendations  

**Question:** What type of reporting includes full documentation in a formal document?  

```
Full formal report
```

**Question:** What should be included in addition to finding name, description, and criticality?  

```
Remediation recommendation
```

---

## Conclusion  

This room provides a solid introduction to the hacker methodology and how each phase builds on the previous one.  

Understanding this workflow is essential before moving to more advanced topics like Active Directory attacks or real-world penetration testing.
