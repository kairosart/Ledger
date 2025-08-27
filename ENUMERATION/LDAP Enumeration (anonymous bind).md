## 🔹 What is LDAP?

- **LDAP** = _Lightweight Directory Access Protocol_
    
- It’s the way you query and interact with **Active Directory (AD)** (users, groups, computers, domain policies, etc.).
    
- AD Domain Controllers listen on **389/tcp (LDAP)** and **636/tcp (LDAP over SSL)**.
    

---

## 🔹 What is an "anonymous bind"?

- Normally you authenticate to LDAP with a domain username + password.
    
- But some ADs (if misconfigured) allow you to **bind without credentials** (anonymous login).
    
- This means anyone can connect and query objects (like a read-only guest).
    

That’s what happened here — your `ldapsearch` worked but returned a **referral** instead of direct data.


## 🔹Running

#Attacking_machine

```
ldapsearch -x -H ldap://10.10.37.10 -b "DC=labyrinth,DC=local"
```

![[LDAP Enumeration (anonymous bind)-20250826200238345.webp]]

### 👉 Explanation

1. The DC accepted your **anonymous bind**.
    
2. Instead of returning full data, it **redirected (referral)** to the **domain FQDN** (`labyrinth.local`).
    
    - This is common in AD — the DC tells you “don’t query me via IP, use the proper domain name”.
        

So the important discovery here is:

- Domain Name = `labyrinth.local`
    
- Hostname = `LABYRINTH`

## 🔹 Why is this useful?

1. You now know the **AD domain name** → important for Kerberos, SMB, RDP, etc.
    
2. With anonymous LDAP access, you may still be able to enumerate:
    
    - Users (`sAMAccountName`, `userPrincipalName`)
        
    - Groups (like `Domain Admins`)
        
    - Policies (password requirements, lockout policy)
        

Even if some attributes are restricted, **knowing the domain structure** helps in brute-forcing, Kerberos attacks (`kerbrute`, `GetNPUsers.py`), or laterally moving with valid creds.

> [!NOTICE]
> ### LDAP (`ldapsearch`) doesn’t know the domain

- When you connect to **LDAP by IP**, it only gives you referrals or errors if you don’t bind correctly.
    
- Without proper **base DN** or credentials, you only get **referrals**, which may include outdated or placeholder names (`labyrinth.local`).
    
- LDAP relies on either:
    
    1. **Correct base DN** (`DC=thm,DC=local`)
        
    2. **DNS to resolve the server’s FQDN** for referrals
        
- If you guessed the wrong domain (`labyrinth.local`), you get **“referral” errors** pointing to a domain that doesn’t exist in your lab.

---

## SMB (`nxc`) knows the “right” domain

- When you connect to **SMB**, the Windows host responds with its **NetBIOS and Active Directory domain** in the initial handshake.
    
- Tools like `nxc` or `crackmapexec` parse this handshake and show the **actual domain name** (`thm.local` in your case).
    
- You don’t need DNS for this — the server tells you directly. 

#Attacking_machine 
Run the following code to know the real domain:
```
nxc smb <MACHINE IP> -u "" -p "" --shares
```

![[LDAP Enumeration (anonymous bind)-20250827120059091.webp]]

### Analysis

1. **SMB is up** on 445 and the host is running Windows 10 / Server 2019.
    
2. Domain name is `thm.local` — note it’s different from `labyrinth.local`**. This explains why your previous Kerberos attempts failed.
    
3. `STATUS_ACCESS_DENIED` means your current session **doesn’t have privileges to enumerate shares** (null session is blocked, or SMB signing prevents it).

## /etc/hosts

- Add this to `/etc/hosts` so your tools resolve the FQDN:
    
    `<MACHINE IP>   `thm.local` LABYRINTH`

## kerbrute  against domain

- Re-run kerbrute against the domain name instead of IP:
-
```
kerbrute userenum --dc 10.10.180.0 -d thm.local /usr/share/seclists/Usernames/top-usernames-shortlist.txt
```

![[LDAP Enumeration (anonymous bind)-20250827120647811.webp]]

You’ve successfully discovered **valid AD usernames** on the domain `thm.local`:

`guest@thm.local 
`administrator@thm.local`

---

### 🔹 What this means

1. `guest` and `administrator` accounts exist  in Active Directory.
    
2. These are **valid accounts** that can be used for further attacks:
    
    - Password spraying
        
    - SMB login
        
    - LDAP queries
        
    - Kerberos attacks (AS-REP roasting, if preauth is disabled)
        
3. Since you now know the  correct domain (`thm.local`),  you can retry LDAP or Kerberos enumeration with these usernames.


---
### LDAP error

However, if you try to run:
```
ldapsearch -x \
 -H ldap://10.10.180.0 \
 -D "thm.local\administrator" \
 -w "123456" \
 -b "DC=thm,DC=local" \
 "(objectClass=user)" \
 sAMAccountName
```

You got an error:
`ldap_bind: Invalid credentials (49) additional info: 80090308: LdapErr: DSID-0C090439, comment: AcceptSecurityContext error, data 52e`

### What it means

- **Error 49 / data 52e** → **invalid username or password** for LDAP bind.
    
- Even though `nxc` confirmed the credentials work for SMB (`administrator:123456`), Windows can have **different restrictions per service**:
    

1. The `administrator` account is **restricted from logging in remotely via LDAP**.
    
2. Guest accounts almost always cannot bind via LDAP.
    
3. SMB and LDAP often have **independent authentication rules**.
    

So, your current creds **cannot be used for LDAP enumeration**.

