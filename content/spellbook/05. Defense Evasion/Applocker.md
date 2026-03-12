# Enumerate applocker rules

Windows applocker can block execution of certain binary, as a blacklist, or allow only some binary, as a whitelist. Supper annoying

```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

# Test policy

This test if every local users can use `cmd.exe`

```powershell
Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone
```