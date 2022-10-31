**Title 

``` powershell

```


**Remove background, set to black**

``` powershell

reg add "HKEY_CURRENT_USER\Control Panel\Desktop" /v WallPaper /t REG_SZ /d " " /f
reg add "HKEY_CURRENT_USER\Control Panel\Colors" /v Background /t REG_SZ /d "0 0 0" /f
```

Remove Mulittask and Search 

``` powershell

reg add "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Search" /v SearchBoxTaskBar /t REG_DWORD /d 0 /f

reg add "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Explorer\MultiTaskingView" /ve /t REG_SZ /f

reg add "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Explorer\MultiTaskingView\AllUpView" /v Enabled /t REG_DWORD /d 0 /f



```
