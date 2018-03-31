_Please_, do not open an issue with a message:
> it does not work

be more prolific

## Check you use latest `mimikatz` version
`mimikatz` official URL is: http://blog.gentilkiwi.com/mimikatz, it links to: https://github.com/gentilkiwi/mimikatz/releases/latest which is a virtual link to the latest release I published.

You can also test _n-1_ (or _n-x_) version from: https://github.com/gentilkiwi/mimikatz/releases.

I'm not responsible for other versions like in `meterpreter` (`mimikatz` or `kiwi` module) or in `PowerShell` (`Invoke-Mimikatz.ps1` script). But I like them :), so don't hesitate to mention me (@gentilkiwi) in the issue, if I can do something, [I will](/PowerShellMafia/PowerSploit/issues/147#issuecomment-224742494)!

## Check you don't have a paranoid "security" product
Sometimes, Anti-virus or HIPS catch some behavior (yeah, it can happen!). Don't hesitate to disable them, or test `mimikatz` in another clean machine (ideally on a virtual machine, up to date).
> Don't ask me why there is no file in the ZIP, or when you've decompressed it.

## Give me informations!
1. What Windows version it is?
   * `Windows 2003` is not a good answer... `Windows 2003 Enterprise R2 SP2 x64 (5.2.3790) - French` is.
   * `Windows 10` is not a good answer... `Windows 10 Enterprise version 1511 x64 (10.0.10586) - French` is.
2. Is this Windows up to date?
   * more or less of course, but if the issue occurred after an update, it's important!
3. What exact version of `mimikatz` are you running ?
   * it's in the header when you start it... `mimikatz 2.1 (x64) built on Jun  8 2016 23:11:47`
   * maybe you'll see yourself that it's better to run `mimikatz` x64 with Windows x64
4. Depending on error/usage, file informations (full version, modification data), by eg: `5.2.3790.4806 - 20/12/2010, 14:38:59`
   * `sekurlsa::*`: `lsasrv.dll`, `msv1_0.dll`, `tspkg.dll`, `wdigest.dll`, `kerberos.dll`, `livessp.dll` & `dpapisrv.dll`
   * `misc::skeleton`: `kdcsvc.dll` & `cryptdll.dll`
   * `lsadump::* /patch`: `lsasrv.dll`, `lsadb.dll` & `samsrv.dll`
   * `crypto::capi` or `crypto::cng`: `rsaenh.dll`, `ncrypt.dll`/`ncryptprov.dll`
   * `event::drop`: `eventlog.dll` or `wevtsvc.dll`
   * `ts::multirdp`: `termsrv.dll`
5. Do you run it with administrator privilege, or `SYSTEM`?
   * really? You know about UAC?
6. How can I reproduce the issue?
7. All information you think important for my understanding of the problem
   * for ``lsadump::dcsync``, forest architecture, domain functionnal level/domain controller version, ...
   * for Kerberos stuff and (over)pass-the-*, how do you access ressources ? separate session, previous tickets, ...
   * ...
8. For compilation issue, what is the Visual Studio version?
   * Do not forget that `mimidrv` can only be built with Windows Driver Kit **7.1** (WinDDK)
   * `mimilove` is for Windows 2000 only, and must be built with Visual Studio 2008 or WinDDK
9. Surprise me!

## Paste the full log file
`mimikatz` can log all input/output to a log file, just type `log` at the prompt.
```
mimikatz # log
Using 'mimikatz.log' for logfile : OK
```
You'll have a nice `mimikatz.log` file in the current directory :) Paste its content in the issue between a code block
<pre>
```
your log
```
</pre>
> Yes, I like pretty issue

## Use cool memes
This will increase your chances that I spend time on it.

![](http://www.mememaker.net/static/images/memes/3610602.jpg)

At least it will make me smile.

## Give me files!

1. For `sekurlsa` stuff, a minidump of the `lsass.exe` process
   * you can get it with Task manager and/or `procdump` (`procdump -accepteula -ma lsass.exe lsass.dmp`)
2. [_Trap for Microsoft_] a minidump of the `lsaiso.exe` process
3. For `dpapi` or `crypto` stuff, files from: https://onedrive.live.com/redir?resid=A352EBC5934F0254%213104
4. For Kerberos stuff, tickets files (usually `*.kirbi` or `*.ccache`)
5. Depending on the error, do not hesitate to push DLL files indicated in [#give-me-informations](#give-me-informations)

It can contain very sensitive data. If it's not from a test machine and you really want to send it to me: encrypt and use mail.

For `benjamin [at] gentilkiwi.com` you can encrypt your mail with S/MIME with this certificate:
```
-----BEGIN CERTIFICATE-----
MIIFBjCCA+6gAwIBAgIQX9t2zc8FxDAjX7I4AC+0dTANBgkqhkiG9w0BAQsFADB1
MQswCQYDVQQGEwJJTDEWMBQGA1UEChMNU3RhcnRDb20gTHRkLjEpMCcGA1UECxMg
U3RhcnRDb20gQ2VydGlmaWNhdGlvbiBBdXRob3JpdHkxIzAhBgNVBAMTGlN0YXJ0
Q29tIENsYXNzIDEgQ2xpZW50IENBMB4XDTE2MDYxMjEwMTUwNVoXDTE3MDYxMjEw
MTUwNVowSjEgMB4GA1UEAwwXYmVuamFtaW5AZ2VudGlsa2l3aS5jb20xJjAkBgkq
hkiG9w0BCQEWF2JlbmphbWluQGdlbnRpbGtpd2kuY29tMIIBIjANBgkqhkiG9w0B
AQEFAAOCAQ8AMIIBCgKCAQEAwakXQyPpwSdYhdtQ6iW//gjc7UvfMzaNB6GWcxsD
fMPyJSzvd/+kXjvgvkYihNLg6d0urlrF+mD75nyGKDyWF3XVnj4p+hiJkoOOG5VH
wE5F3L11iGsQ/CdbX3IUPuOCSnUeiWm68Du4M2tONtTZeT34ledsp3Fsfu8imDEP
b/YyPP9JK/QYnPB4FuJRcDSn1uwHkJJhdJhWsoG+GjOZCOrleSlWZWINF+Seu8PQ
pfHBKu45zz4EYJ09RAavjgmHOskqZCIKPcgJp8Q+H/8nl07IaS9/S5j18Z95iZ+j
IGV+UF7cjClLZwuH8j9GxP3pOEAGlzBiTe2Bc5zrpYfc1wIDAQABo4IBuzCCAbcw
DgYDVR0PAQH/BAQDAgSwMB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcDBDAJ
BgNVHRMEAjAAMB0GA1UdDgQWBBQIydWKebK8yknYaQPVv4KMIhCa8DAfBgNVHSME
GDAWgBQkgWw5Yb5JD4+3G0YrySi1J0htaDBvBggrBgEFBQcBAQRjMGEwJAYIKwYB
BQUHMAGGGGh0dHA6Ly9vY3NwLnN0YXJ0c3NsLmNvbTA5BggrBgEFBQcwAoYtaHR0
cDovL2FpYS5zdGFydHNzbC5jb20vY2VydHMvc2NhLmNsaWVudDEuY3J0MDgGA1Ud
HwQxMC8wLaAroCmGJ2h0dHA6Ly9jcmwuc3RhcnRzc2wuY29tL3NjYS1jbGllbnQx
LmNybDAiBgNVHREEGzAZgRdiZW5qYW1pbkBnZW50aWxraXdpLmNvbTAjBgNVHRIE
HDAahhhodHRwOi8vd3d3LnN0YXJ0c3NsLmNvbS8wRwYDVR0gBEAwPjA8BgsrBgEE
AYG1NwECBTAtMCsGCCsGAQUFBwIBFh9odHRwczovL3d3dy5zdGFydHNzbC5jb20v
cG9saWN5MA0GCSqGSIb3DQEBCwUAA4IBAQCv2FoU5ntOFJo09Q07u9YeWrPxpF6H
I5XNrr/3l/mmfnZzHdqIhq4WrYzFFO5vI+0BUaE9Pb2qZIcQYRJR4EgKoaQUEGKP
0IcsAjaI9rGyODntogDlV42VSIRxWyTtdi0r/enmJmulzlDkEzkbuEAfhqv7LVVI
Fd2USFgt0UX7ts7YfcbVuIdcZe4g0TWbd7mhtxc84lV4skzBop0Juznl+YhdbIoU
aEGq8Ba1M0HkAXAcIBj0sCKLEEeXQ+Kn2QPpAMvwuZqpSampxY0HDSId3jp0Sf6d
12vLc/1jLxrExJZHIlk0AA9T+OaZB3KsNtljYIWRkF/Btz2v9xMnmRqA
-----END CERTIFICATE-----
```

I'm not a soothsayer, keep in mind that without some of these elements, I can not help you.

## Be logic
1. `mimikatz` will not extract private key from a smartcard/hsm/token.
2. `mimikatz` will not get passwords from memory when they are not in memory (default from 8.1 - [#40](../issues/40#issuecomment-220830921))

## Be cool
I'm only a kiwi, I code for my pleasure and when I've time. If you want a SLA it will cost you a lots of fruits.