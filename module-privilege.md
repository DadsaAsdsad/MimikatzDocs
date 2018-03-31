This module provides some commands to manipulate privilege on `mimikatz` process.

## debug
Ask for `debug` privilege for `mimikatz` process.

> The debug privilege allows someone to debug a process that they wouldn’t otherwise have access to. For example, a process running as a user with the debug privilege enabled on its token can debug a service running as local system.  
_from:_ http://msdn.microsoft.com/library/windows/hardware/ff541528.aspx

```
mimikatz # privilege::debug
Privilege '20' OK
```
**Remark:** `ERROR kuhl_m_privilege_simple ; RtlAdjustPrivilege (20) c0000061` means that the required privilege is not held by the client (mostly you're not an administrator :smirk:)