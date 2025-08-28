
#RDP_Connection

You needd to know what privileges has `SUSANNA_MCKNIGHT`.
On a command prompt run:

```
whoami /ALL
```

![[User's privileges-20250828121752118.webp]]

The most interesting privilege is BUILTIN\Certificate Service DCOM Access.

## BUILTIN\Certificate Service DCOM Access

## 🔹 What it is

- It’s a **local group** created when **AD CS** is installed on a Windows server.
    
- Members of this group are allowed to **remotely access the Certificate Services DCOM interface**.
    
- In other words, it controls which accounts can request and interact with **certificate services** over DCOM/RPC.
    

---

## 🔹 Why it matters

- If an account is a member of this group, it can remotely communicate with the **Certificate Authority (CA)** via DCOM.
    
- In a default, properly configured AD CS environment → this is usually just administrative/legitimate service accounts.
    
- In a misconfigured environment → it can be abused in attacks such as:
    
    - **ESC1 / ESC8 attacks (ADCS abuse)** → where certificates can be enrolled and used for authentication as other users (e.g. domain admin impersonation).
        
    - **Relay attacks** → since certificate enrollment can sometimes be done with only low-priv privileges.
        

---

## 🔹 What to check as an attacker

If you see `BUILTIN\Certificate Service DCOM Access` during enumeration:

1. Run **Certify** or **Certipy** to enumerate AD CS templates:
    
    `certipy find -u USER -p PASS -dc-ip <DC-IP>`
    
2. Look for vulnerable templates (`ENROLLEE_SUPPLIES_SUBJECT`, no manager approval, etc.).
    
3. If exploitable, you can request a certificate as **any user** and use it for Kerberos/LDAP logon.
    

---

✅ **Summary:**

- `BUILTIN\Certificate Service DCOM Access` = group that controls remote access to AD CS DCOM.
    
- Membership here means the user can interact with the CA.
    
- On its own, it’s not privilege escalation — but **combined with weak certificate templates**, it’s a **big attack surface** (AD CS exploitation).


## Running

#Attacking_machine 

```
certipy find -u 'SUSANNA_MCKNIGHT' -p 'CHANGEME2023!' -dc-ip <MACHINE IP> -vulnerable -stdout

```

