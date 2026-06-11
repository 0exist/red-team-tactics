# red-team-tactics
Red team and penetration testing notes focused on attack decisions and the reasoning behind tactical choices.

### Fast port enumeration
```bash
TARGET=192.168.98.2; (echo 5985; grep -v '^#' /usr/share/nmap/nmap-services | sort -rk3 | head -n 1000 | awk '{print $2}' | cut -d/ -f1) | awk '!seen[$0]++' | xargs -P8 -I{} bash -c 'for i in 1 2 3; do nc -vz -w4 '"$TARGET"' {} 2>&1 | grep -m1 "succeeded!" && exit 0; done'
```
⚡ Add proxychains before nc if using a proxy tunnel  
⚡ Port 5985 is manually included because it’s not in nmap’s top 1000 ports  
▶️ [Video explanation](https://www.youtube.com/watch?v=Jv-MDAuRcHs&t=37s)

More details to be added in the future, stay tuned
