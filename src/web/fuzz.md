# Fuzzing

## Wfuzz

```
wfuzz -u http://URL/page.pgp?code=FUZZ -w /usr/share/seclists/Fuzzing/special-chars.txt

--hc 404 (ignore 404 results)
--hl=2 (ignore length 2 results)
```

### Post data
```
wfuzz -c -w /wordlist.txt -d "username=FUZZ&password=password" --hs "No acount found with that username" http://IP
```

### FFuf
```
ffuf -w valid_usernames.txt:W1,/usr/share/wordlists/SecLists/Passwords/Common-Credentials/10-million-password-list-top-100.txt:W2 -X POST -d "username=W1&password=W2" -H "Content-Type: application/x-www-form-urlencoded" -u http://MACHINE_IP/customers/login -fc 200
```