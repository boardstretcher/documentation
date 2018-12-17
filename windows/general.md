**reset winsock**
```
netsh winsock reset
reboot
```

**Connecting to RDP session (no license server found)**
```
mstsc.exe /admin
```

**DNS zone transfer**
http://www.windowsnetworking.com/kbase/WindowsTips/WindowsXP/AdminTips/Network/nslookupandDNSZoneTransfers.html
```
set d2
set type=A
ls -d dev.local > somefile.txt
```
