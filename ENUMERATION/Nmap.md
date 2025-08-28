Here’s a **single one-liner** that will:

1. Run a **fast full-port scan**.
```
nmap <MACHINE IP>
```

![[Nmap-20250828103146229.webp]]

2. Extract the **open ports**.

```
sudo nmap -sV -sC -v <MACHINE IP>
```
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-08-28 08:51:52Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: thm.local0., Site: Default-First-Site-Name)
|_ssl-date: 2025-08-28T08:52:44+00:00; +1s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:labyrinth.thm.local, DNS:thm.local, DNS:THM
| Issuer: commonName=thm-LABYRINTH-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-05-12T07:32:36
| Not valid after:  2024-05-11T07:32:36
| MD5:   eae1:9bc6:ffbf:ac19:f750:22bd:7186:943a
|_SHA-1: 5bd6:40fd:76e2:d5ab:3909:5bcc:7a4f:4f4c:f7c6:2e34
443/tcp  open  ssl/http      Microsoft IIS httpd 10.0
|_ssl-date: 2025-08-28T08:52:44+00:00; +1s from scanner time.
|_http-server-header: Microsoft-IIS/10.0
| ssl-cert: Subject: commonName=thm-LABYRINTH-CA
| Issuer: commonName=thm-LABYRINTH-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-05-12T07:26:00
| Not valid after:  2028-05-12T07:35:59
| MD5:   c249:3bc6:fd31:f2aa:83cb:2774:bc66:9151
|_SHA-1: 397a:54df:c1ff:f9fd:57e4:a944:00e8:cfdb:6e3a:972b
| tls-alpn: 
|_  http/1.1
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: thm.local0., Site: Default-First-Site-Name)
|_ssl-date: 2025-08-28T08:52:44+00:00; +1s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:labyrinth.thm.local, DNS:thm.local, DNS:THM
| Issuer: commonName=thm-LABYRINTH-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-05-12T07:32:36
| Not valid after:  2024-05-11T07:32:36
| MD5:   eae1:9bc6:ffbf:ac19:f750:22bd:7186:943a
|_SHA-1: 5bd6:40fd:76e2:d5ab:3909:5bcc:7a4f:4f4c:f7c6:2e34
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: thm.local0., Site: Default-First-Site-Name)
|_ssl-date: 2025-08-28T08:52:44+00:00; +1s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:labyrinth.thm.local, DNS:thm.local, DNS:THM
| Issuer: commonName=thm-LABYRINTH-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-05-12T07:32:36
| Not valid after:  2024-05-11T07:32:36
| MD5:   eae1:9bc6:ffbf:ac19:f750:22bd:7186:943a
|_SHA-1: 5bd6:40fd:76e2:d5ab:3909:5bcc:7a4f:4f4c:f7c6:2e34
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: thm.local0., Site: Default-First-Site-Name)
|_ssl-date: 2025-08-28T08:52:44+00:00; +1s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:labyrinth.thm.local, DNS:thm.local, DNS:THM
| Issuer: commonName=thm-LABYRINTH-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-05-12T07:32:36
| Not valid after:  2024-05-11T07:32:36
| MD5:   eae1:9bc6:ffbf:ac19:f750:22bd:7186:943a
|_SHA-1: 5bd6:40fd:76e2:d5ab:3909:5bcc:7a4f:4f4c:f7c6:2e34
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2025-08-28T08:52:44+00:00; +1s from scanner time.
| rdp-ntlm-info: 
<font color="#ff0000">|   Target_Name: THM</font>
<font color="#ff0000">|   NetBIOS_Domain_Name: THM</font>
<font color="#ff0000">|   NetBIOS_Computer_Name: LABYRINTH</font>
<font color="#ff0000">|   DNS_Domain_Name: thm.local</font>
<font color="#ff0000">|   DNS_Computer_Name: labyrinth.thm.local</font>
|   Product_Version: 10.0.17763
|_  System_Time: 2025-08-28T08:52:35+00:00
| ssl-cert: Subject: commonName=labyrinth.thm.local
| Issuer: commonName=labyrinth.thm.local
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-08-27T08:22:14
| Not valid after:  2026-02-26T08:22:14
| MD5:   bbbe:c7cc:e502:387f:57c8:02f0:a5bb:deee
|_SHA-1: d61b:cfde:a46f:093f:343a:8e18:a3df:7ad1:3574:3634
Service Info: Host: LABYRINTH; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-08-28T08:52:36
|_  start_date: N/A

## Rows on red explanation

#### 🔹 `Target_Name: THM`

- This is the **Kerberos/NTLM realm name** the server presents.
    
- Often the same as the **NetBIOS domain name**, but sometimes shortened.
    

---

#### 🔹 `NetBIOS_Domain_Name: THM`

- The **old Windows domain name** (15 chars max).
    
- Used in legacy authentication (NTLM challenge/response).
    
- Equivalent to the short domain you’d type in `THM\username`.
    

Example:

`THM\administrator`

---

#### 🔹 `NetBIOS_Computer_Name: LABYRINTH`

- The **machine’s NetBIOS hostname**.
    
- Usually how the computer would appear in an older Windows workgroup.
    
- In this case, the host machine is called **LABYRINTH**.
    

---

#### 🔹 `DNS_Domain_Name: thm.local`

- The **Active Directory domain in FQDN form**.
    
- This is the **real domain you should use for LDAP, Kerberos, etc.**
    

Example:

`administrator@thm.local`

---

#### 🔹 `DNS_Computer_Name: labyrinth.thm.local`

- The **fully qualified domain name (FQDN)** of the host.
    
- Equivalent to:
    
    - Hostname: `labyrinth`
        
    - Domain: `thm.local`
        
- Useful for RDP, SMB, and Kerberos.



That’s consistent with a **Domain Controller (DC)** hosting AD services.

**Next step:** [[NXC]]
