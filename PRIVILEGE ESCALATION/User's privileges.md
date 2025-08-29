
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
certipy find -u 'SUSANNA_MCKNIGHT' -p 'CHANGEME2023!' -dc-ip <MACHINE IP> -vulnerable
```

You'll get a txt and a json files.

## Certificate Authorities
  0
    CA Name                             : thm-LABYRINTH-CA
    DNS Name                            : labyrinth.thm.local
    Certificate Subject                 : CN=thm-LABYRINTH-CA, DC=thm, DC=local
    Certificate Serial Number           : 5225C02DD750EDB340E984BC75F09029
    Certificate Validity Start          : 2023-05-12 07:26:00+00:00
    Certificate Validity End            : 2028-05-12 07:35:59+00:00
    Web Enrollment                      : Disabled
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Permissions
      Owner                             : THM.LOCAL\Administrators
      Access Rights
        ManageCertificates              : THM.LOCAL\Administrators
                                          THM.LOCAL\Domain Admins
                                          THM.LOCAL\Enterprise Admins
        ManageCa                        : THM.LOCAL\Administrators
                                          THM.LOCAL\Domain Admins
                                          THM.LOCAL\Enterprise Admins
        Enroll                          : THM.LOCAL\Authenticated Users
## Certificate Templates

Within the "Certificate Templates" section of the `certipy find` output, vulnerable templates are clearly marked, with "ESC1" listed under the `[!] Vulnerabilities` heading.
### 0
    Template Name                       : ServerAuth
    Display Name                        : ServerAuth
    Certificate Authorities             : thm-LABYRINTH-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Enrollment Flag                     : None
    Private Key Flag                    : 16842752
    Extended Key Usage                  : Client Authentication
                                          Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Permissions
      Enrollment Permissions
        Enrollment Rights               : THM.LOCAL\Domain Admins
                                          THM.LOCAL\Domain Computers
                                          THM.LOCAL\Enterprise Admins
                                          THM.LOCAL\Authenticated Users
      Object Control Permissions
        Owner                           : THM.LOCAL\Administrator
        Write Owner Principals          : THM.LOCAL\Domain Admins
                                          THM.LOCAL\Enterprise Admins
                                          THM.LOCAL\Administrator
        Write Dacl Principals           : THM.LOCAL\Domain Admins
                                          THM.LOCAL\Enterprise Admins
                                          THM.LOCAL\Administrator
        Write Property Principals       : THM.LOCAL\Domain Admins
                                          THM.LOCAL\Enterprise Admins
                                          THM.LOCAL\Administrator
###### [!] Vulnerabilities
      ESC1                              : 'THM.LOCAL\\Domain Computers' and 'THM.LOCAL\\Authenticated Users' can enroll, enrollee supplies subject and template allows client authentication
### 1
    Template Name                       : Computer2
    Display Name                        : Computer2
    Enabled                             : False
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Enrollment Flag                     : None
    Private Key Flag                    : 16842752
    Extended Key Usage                  : Server Authentication
                                          Client Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Permissions
      Enrollment Permissions
        Enrollment Rights               : THM.LOCAL\Domain Admins
                                          THM.LOCAL\Domain Computers
                                          THM.LOCAL\Enterprise Admins
                                          THM.LOCAL\Authenticated Users
      Object Control Permissions
        Owner                           : THM.LOCAL\Administrator
        Write Owner Principals          : THM.LOCAL\Domain Admins
                                          THM.LOCAL\Enterprise Admins
                                          THM.LOCAL\Administrator
        Write Dacl Principals           : THM.LOCAL\Domain Admins
                                          THM.LOCAL\Enterprise Admins
                                          THM.LOCAL\Administrator
        Write Property Principals       : THM.LOCAL\Domain Admins
                                          THM.LOCAL\Enterprise Admins
                                          THM.LOCAL\Administrator
###### [!] Vulnerabilities
      ESC1                              : 'THM.LOCAL\\Domain Computers' and 'THM.LOCAL\\Authenticated Users' can enroll, enrollee supplies subject and template allows client authentication


---

## 🚩 Vulnerability: ESC1

**ESC1** = **Misconfigured certificate templates that allow privilege escalation**.

The dangerous combination is:

1. **Enrollee supplies subject** → attacker can request a certificate for _any account_ (e.g., `Administrator` or a domain controller service account).
    
2. **Client Authentication EKU** → certificate can be used for authentication (Kerberos/NTLM).
    
3. **Enrollment rights for low-privileged groups (e.g., Authenticated Users, Domain Computers)** → anyone in the domain can request such a certificate.
    

Effect:

- A normal user could request a certificate claiming to be `Administrator`.
    
- Use that certificate to obtain a **Kerberos TGT** for Administrator (`PKINIT`).
    
- Result: **Domain Admin compromise**.

**Next step:** [[Exploiting ESC1 vulnerable template]]
