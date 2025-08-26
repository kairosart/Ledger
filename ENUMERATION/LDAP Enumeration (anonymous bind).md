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


## Running

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


---

## /etc/hosts

- Add this to `/etc/hosts` so your tools resolve the FQDN:
    
    `10.10.37.10   labyrinth.local labyrinth`
    
- Re-run ldapsearch against the domain name instead of IP:
    
    `ldapsearch -x -H ldap://labyrinth.local -b "DC=labyrinth,DC=local"`
    
- If anonymous access is still partially open, you’ll start seeing accounts and groups.

## What the LDAP output means

`result: 10 Referral ref: ldap://labyrinth.local/DC=labyrinth,DC=local`

- `result: 10 Referral` → The LDAP server didn’t give you objects, it told you to look elsewhere.
    
- **Referral** means: _“Don’t query me by IP, use the proper domain controller address.”_
    
- The referral is pointing to:
    
    `ldap://labyrinth.local/DC=labyrinth,DC=local`
    
- That reveals two important things:
    
    - The **Active Directory domain name**: `labyrinth.local`
        
    - The **base DN** for queries: `DC=labyrinth,DC=local`
        

---

### ⚡ Why this matters

Even though you only got a referral, you now know:

1. ✅ The AD domain name → `labyrinth.local`
    
2. ✅ The server’s hostname (from earlier nmap) → `LABYRINTH`
    
3. ✅ LDAP is running, and **anonymous bind works** (at least partially).
    

This is the first real foothold into enumerating Active Directory.