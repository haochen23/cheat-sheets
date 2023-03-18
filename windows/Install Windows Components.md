## Install IIS
```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

## .Net framework

### determine installed .Net framework version
```powershell
(Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full").Release -ge 394802

(Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full").Version
```
