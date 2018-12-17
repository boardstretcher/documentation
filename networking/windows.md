## general troubleshooting
**register query**
```
reg query "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings" | find /i "proxyserver"
```
**basic troubleshooting commands**
```
netstat -ban
route print
tracert -d
ping -n 1
ipconfig /all
```
