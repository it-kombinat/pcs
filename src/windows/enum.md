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

#### Net User Domain

````
net user /domain
````
Find domain and local admin
```
net localgroup administrators
```

#### CMDLETS
##### Get-ADUser
The command reveals that the user is a member of the Audit Share group, and also that the logon script MapAuditDrive.vbs is assigned to this account. 
Active Directory logon scripts are saved in the NETLOGON share by default)
````
*Evil-WinRM* PS C:\Users\s.smith\Desktop> Get-ADUser -identity s.smith -properties * 
````
Filter - Parameter that allows more controll over enumeration and use the Format-Table.
```
Get-ADUser -Filter 'Name -like "*stevens*"' -Server ad.server.com | Format-Table Name,SamAccountName -A
```

If we wanted to, for example, perform a password spraying attack without locking out accounts, we can use this to enumerate accounts that have a badPwdCount that is greater than 0, to avoid these accounts in our attack:
```
 Get-ADObject -Filter 'badPwdCount -gt 0' -Server ad.server.com
```
##### Further cmdlets
Get-ADGroup, Get-ADGroupMember, Get-ADObject, Get-ADDomain