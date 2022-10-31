
IP Address
``` PowerShell
New-NetIPAddress -InterfaceIndex 12 -IPAddress 192.168.0.1 -PrefixLength 24 -DefaultGateway 192.168.0.5
Remove-NetIPAddress -IPAddress 192.168.0.1 -DefaultGateway 192.168.0.5
```

