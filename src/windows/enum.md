## Reconnaissance

### Windows

`dir /s flag.txt` to find files in current directory and subdirectories named flag.txt.


### Active Directory

#### Tool

* [wadcoms.github.io](https://wadcoms.github.io)
* [windapsearch](https://github.com/ropnop/windapsearch)

#### Ldapsearch

````
ldapsearch -h <host> -p 389 -x -b "dc=cascade,dc=local" "(&(objectClass=user)(sAMAccountName=userid))" 
````

#### Get-ADUser
The command reveals that the user is a member of the Audit Share group, and also that the logon script MapAuditDrive.vbs is assigned to this account. 
Active Directory logon scripts are saved in the NETLOGON share by default)
````
*Evil-WinRM* PS C:\Users\s.smith\Desktop> Get-ADUser -identity s.smith -properties * 
````
