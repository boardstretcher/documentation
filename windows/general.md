**basic system maintenance**
```
sfc /scannow
dism /online /cleanup-image /restorehealth
```

**convert output to html**
```
Get-Service | ConvertTo-HTML -Property Name, Status > C:\Services.htm
```

**basic commands**
```
Get-Item a*
Copy-Item "C:\Services.htm" -Destination "C:\MyData\MyServices.txt"
Remove-Item "C:\MyData\MyServices.txt"

Set-Variable -Name "desc" -Value "A Description"
Get-Variable -Name "desc"

Get-Process *firefox*
Start-Process
Stop-Process

Get-Service *firefox*
Start-service
Stop-service

#windows cat
Get-Content "C:\Services.htm" -TotalCount 50
```

**scrape webpage**
```
(Invoke-WebRequest -Uri "https://docs.microsoft.com").Links.Href
```

**get events in powershell**
```
Invoke-Command -ScriptBlock {Get-EventLog Application -newest 50}
```

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

**get recent updates**
```
gwmi win32_quickfixengineering |sort installedon -desc
```

**temporarily allow bypass**
```
Set-ExecutionPolicy -Scope Process -ExecutionPolicy  ByPass
```

**update from commandline**
```
Install-Module -Name PSWindowsUpdate
get-wulist
Install-WindowsUpdate -AcceptAll
```
