## Downgrade Powershell

> Powershell event logging is available on Powershell >3.0. So we can call Powershell version 2.0 or older. If successful, our commands will not be logged in Event Viewer.

```powershell
## Get current powershell version
Get-host
## Use PS version 2
powershell.exe -version 2
```

## Powershell execution policy bypass

```powershell
powershell -ep bypass
```

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

## Powershell Constrained Language Mode

PowerShell [Constrained Language Mode](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/) locks down many of the features needed to use PowerShell effectively, such as blocking COM objects, only allowing approved .NET types, XAML-based workflows, PowerShell classes, and more

```powershell
$ExecutionContext.SessionState.LanguageMode
```