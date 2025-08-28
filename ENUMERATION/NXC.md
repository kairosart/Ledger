The first thing I did is enumerating the domain controller for users and policies. To accomplish this, you can use `netexec` and query LDAP which allows you to query user information.

#Attacking_machine

```
nxc ldap <MACHINE IP> -u "a" -p "" --users
```

### 🔍 Explanation

- `nxc` → this is **NetExec** (the maintained fork of CrackMapExec). It’s a post-exploitation enumeration tool.
    
- `ldap` → tells `nxc` to use the **LDAP protocol (389/tcp)** instead of SMB, WinRM, etc. LDAP is used to query Active Directory objects (users, groups, computers).
    
- `<MACHINE IP>` → the **target IP** (probably the DC or a Windows server joined to the domain).
    
- `-u "a"` → username `a`.
    
    - This looks like a "null session" test (a common trick is using fake usernames like `a`, `test`, or `guest`).
        
    - Sometimes DCs allow **anonymous (null) binds** that still leak user info.
        
- `-p ""` → empty password.
    
    - This means you’re testing authentication with a blank password (guest-style login).
        
- `--users`*→ specific **NetExec module** that enumerates all **domain users** via LDAP.
    
    - It pulls attributes like:
        
        - `sAMAccountName` (login name)
            
        - `displayName`
            
        - `description`
            
        - `lastLogon`
            
        - account status (enabled/disabled, etc.)

### ⚡ What happens when you run it

- NetExec tries to **bind to LDAP** on `<MACHINE IP>` with the creds:
    
    `username: a password: (empty)`
    
- If the server allows **anonymous/guest LDAP binds**, it will dump the list of **all domain users**.
    
- If not, you’ll see something like:
    
    `LDAP <MACHINE IP> 389 LAB  [-] a: Invalid credentials`

### 🧠 Why this is useful

- In AD pentests / CTFs, LDAP often leaks **usernames** even without valid creds.
    
- Once you have usernames, you can:
    
    - **Password spray** (e.g., test weak/default passwords like `123456`, `Password1!`).
        
    - Try **Kerberos enumeration** (`GetNPUsers.py`, `kerbrute`).
        
    - Pivot into **AS-REP roasting** or **Kerberoasting**.

## Results

You'll get a long list of users. There are two interesting ones:

LDAP        10.10.41.122    389    LABYRINTH        IVY_WILLIS    2023-05-30 12:30:55 0       Please change it: CHANGEME2023!
LDAP        10.10.41.122    389    LABYRINTH        SUSANNA_MCKNIGHT              2023-07-05 15:11:32 0       Please change it: CHANGEME2023! 

### 🔍 Field by field

- **`LDAP`** → the protocol used.
    
- **`10.10.41.122`** → target host (likely the Domain Controller).
    
- **`389`** → LDAP port.
    
- **`LABYRINTH`** → the domain/realm name (NetBIOS).
    
- **`IVY_WILLIS`** → the **username** (sAMAccountName) of the domain account.
    
- **`2023-05-30 12:30:55`** → last logon timestamp.
    
- **`0`** → userAccountControl flag (likely means "enabled" account).
    
- **`Please change it: CHANGEME2023!`** → this is the **description field** of the account.

### 🔍 Field by field

- **`LDAP`** → the protocol used.
    
- **`10.10.41.122`** → target host (likely the Domain Controller).
    
- **`389`** → LDAP port.
    
- **`LABYRINTH`** → the domain/realm name (NetBIOS).
    
- **`IVY_WILLIS`** → the **username** (sAMAccountName) of the domain account.
    
- **`2023-05-30 12:30:55`** → last logon timestamp.
    
- **`0`** → userAccountControl flag (likely means "enabled" account).
    
- **`Please change it: CHANGEME2023!`** → this is the **description field** of the account.

**Next step:** [[Attack Path]]