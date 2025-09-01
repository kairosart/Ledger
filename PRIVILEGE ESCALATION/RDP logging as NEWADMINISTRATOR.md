
As you already have a new administrator account with all privileges, now you can connect via RDP.

#Attacking_machine 

```
xfreerdp3 /u:NEWADMINISTRATOR@thm.local /p:'P@ssw0rd123!' /v:<MACHINE IP>
```

- Go to the Administrator desktop where you'll find a file called `root`.

- Open it and you'll get the second flag.

> [!Question] What is the root flag?
>`THM{THE_BYPASS_IS_CERTIFIED!}`

