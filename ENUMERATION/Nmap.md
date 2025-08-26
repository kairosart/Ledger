Here’s a **single one-liner** that will:

1. Run a **fast full-port scan**.

2. Extract the **open ports**.

3. Run a **deep enumeration** only on those ports.

```
nmap -p- --min-rate 10000 -T4 -v <MACHINE IP> -oG all_ports.txt && ports=$(grep -oP '\d+/open' all_ports.txt | cut -d'/' -f1 | tr '\n' ',' | sed 's/,$//') && nmap -p$ports -sC -sV -A -T4 <MACHINE IP> -oN full_enum.txt
```

## What it does:

- First `nmap -p- ...` → scans all 65,535 ports quickly.
- Saves results to `all_ports.txt`.
- Extracts open ports into `$ports`.
- Runs a detailed scan (`-sC -sV -A`) **only on those ports**.
- Saves detailed output in `full_enum.txt`.

## Results

- **Open services**:
    
    - `53/tcp` → DNS
        
    - `88/tcp` → Kerberos
        
    - `135, 139, 445/tcp` → RPC, SMB
        
    - `389, 636/tcp` → LDAP (unencrypted + SSL)
        
    - `3268, 3269/tcp` → Global Catalog LDAP
        
    - `3389/tcp` → RDP
        
    - `80, 443, 593/tcp` → HTTP/HTTPS/HTTP-RPC
        
- **Nmap OS guess**: Windows Server 2019 / 2016 / 2012 (likely a DC)
    
- **Service Info**: Hostname = `LABYRINTH`


That’s consistent with a **Domain Controller (DC)** hosting AD services.

**Next step:** [[LDAP Enumeration (anonymous bind)]]
