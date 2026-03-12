# Check defender status

If `RealTimeProtectionEnabled` is set to `True`, then it is on

```powershell
Get-MpComputerStatus
```

```cmd
sc query windefend
```

# Disable defender (need admin)

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
```

# Check exclusion (no priv)

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Windows Defender/Operational" -FilterXPath "*[System[(EventID=5007)]]" | Where-Object { $_.Message -like "*Exclusion*"} | Select-Object Message | FL
```