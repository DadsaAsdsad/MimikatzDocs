`mimikatz` is a tool I've made to learn C and make somes experiments with Windows security.

It's well known to [extract plaintexts passwords, hash, PIN code and kerberos tickets](module-~-sekurlsa) from memory.  
`mimikatz` can also perform [pass-the-hash](module-~-sekurlsa#pth), [pass-the-ticket](module-~-kerberos#ptt), build [Golden tickets](module-~-kerberos#golden), play with certificates or private keys, vault, ... _maybe make coffee?_

Its symbol/icon is a **kiwi**, sometimes the **animal**, but mostly the **fruit**!
```
  .#####.   mimikatz 2.0 alpha (x64) release "Kiwi en C" (Apr 26 2014 00:25:11)
 .## ^ ##.
 ## / \ ##  /* * *
 ## \ / ##   Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 '## v ##'   http://blog.gentilkiwi.com/mimikatz             (oe.eo)
  '#####'                                    with  14 modules * * */
```
### How can you get it?
* sources (Visual Studio solution) on [GitHub/mimikatz](../) - see [GitHub/mimikatz/README # build](../blob/master/README.md#build)
* binaries are availables on [GitHub/mimikatz/releases](../releases/latest)

## Basics
`mimikatz` comes in two flavors: `x64` or `Win32`, depending on your windows version (32/64 bits).  
`Win32` flavor cannot access 64 bits process memory (like `lsass`), but can open 32 bits minidump under Windows 64 bits.  
Some operations need administrator privileges, or `SYSTEM` token, _so be aware of `UAC` from Vista version_.

After launching `mimikatz`:
```
  .#####.   mimikatz 2.0 alpha (x64) release "Kiwi en C" (Apr 26 2014 00:25:11)
 .## ^ ##.
 ## / \ ##  /* * *
 ## \ / ##   Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 '## v ##'   http://blog.gentilkiwi.com/mimikatz             (oe.eo)
  '#####'                                    with  14 modules * * */


mimikatz #
````
... you have the command prompt `mimikatz #`, you can type instructions like `exit`, `cls`, `crypto::certificates`

Instructions can be in the form: `modulename::commandname arguments...`, eg:
 * `kerberos::tgt`
 * `crypto::certificates /systemstore:local_machine /store:my /export`
 * `cls`

see [Module](#modules) section below for others.  
_commands from `standard` module can be typed without `modulename`; `cls` is the same as `standard::cls` (see [module ~ standard](module-~-standard))_

You can quit `mimikatz` with `exit` command.  
For remote execution, see [howto ~ remote execution](howto-~-remote-execution)

### Command line
You can pass instructions on `mimikatz` command line, those with arguments/spaces must be quoted.

```
C:\security\mimikatz\x64>mimikatz log version "crypto::certificates /systemstore:local_machine" exit

  .#####.   mimikatz 2.0 alpha (x64) release "Kiwi en C" (Apr 26 2014 00:25:11)
 .## ^ ##.
 ## / \ ##  /* * *
 ## \ / ##   Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 '## v ##'   http://blog.gentilkiwi.com/mimikatz             (oe.eo)
  '#####'                                    with  14 modules * * */


mimikatz(commandline) # log
Using 'mimikatz.log' for logfile : OK

mimikatz(commandline) # version

mimikatz 2.0 alpha (arch x64)
NT     -  Windows NT 6.1 build 7601 (arch x64)

mimikatz(commandline) # crypto::certificates /systemstore:local_machine
 * System Store  : 'local_machine' (0x00020000)
 * Store         : 'My'

 0. example.nirvana.local
        Key Container  : example.nirvana.local
        Provider       : Microsoft Software Key Storage Provider
        Type           : CNG Key (0xffffffff)
        Exportable key : NO
        Key size       : 2048

mimikatz(commandline) # exit
Bye!
```
Instructions from command line are marked with `(commandline)` on the prompt.

## Modules
* [standard](module-~-standard)
* [privilege](module-~-privilege)
* [crypto](module-~-crypto)
* [sekurlsa](module-~-sekurlsa)
* [kerberos](module-~-kerberos)
* [lsadump](module-~-lsadump)
* [vault](module-~-vault)
* [token](module-~-token)
* [event](module-~-event)
* [ts](module-~-ts)
* [process](module-~-process)
* [service](module-~-service)
* [net](module-~-net)
* [misc](module-~-misc)
* [library `mimilib`](library-~-mimilib)
* [driver `mimidrv`](driver-~-mimidrv)

## About me
I'm a kiwi.

## History
`mimikatz` is now **2.0**, but is born in 2007, it was known by other names:
* `kdll` ; a simple DLL injector
* `kdllpipe` ; first version to accomplish Pass-The-Hash, with interaction on a named pipe
* `katz` ;
* `mimikatz` !

I started to code it for some reasons:
* improve my knowledge, especially in C/C++ for Windows ;
* explain security concepts ;
* prove to Microsoft that sometimes they must change old habits.

## External resources
### Some amazing alternative versions of `mimikatz`, w00t!

* Meterpreter extension for `mimikatz 1.0` by **[Ben Campbell](https://twitter.com/Meatballs__)**   
  [Meterpreter](https://github.com/rapid7/meterpreter/tree/master/source/extensions/mimikatz) & [Metasploit](https://github.com/rapid7/metasploit-framework/tree/master/lib/rex/post/meterpreter/extensions/mimikatz)

* Meterpreter extension for `mimikatz 2.0` by **[Oliver Reeves](https://twitter.com/TheColonial)**  
  [Meterpreter](https://github.com/rapid7/meterpreter/tree/master/source/extensions/kiwi) & [Metasploit](https://github.com/rapid7/metasploit-framework/tree/master/lib/rex/post/meterpreter/extensions/kiwi)

* DLL reflection in PowerShell by **[Joseph Bialek](https://twitter.com/JosephBialek)**  
  [Script](https://github.com/clymb3r/PowerShell/tree/master/Invoke-Mimikatz) & [Information](http://clymb3r.wordpress.com/2013/04/09/modifying-mimikatz-to-be-loaded-using-invoke-reflectivedllinjection-ps1)

* Volatility plugin by **[Francesco Picasso](https://plus.google.com/+francescopicasso)**  
  [Plugin](http://blog.digital-forensics.it/2014/03/et-voila-le-mimikatz-offline.html) & [Information](http://blog.digital-forensics.it/2014/03/mimikatz-offline-addendum_28.html)

### Some ressources _inspired_ by my work
* `wce` (cleartext passwords part) by **[Hernan Ochoa](https://twitter.com/hernano)** @ [Amplia security](http://www.ampliasecurity.com)  
  [WCE faq](http://www.ampliasecurity.com/research/wcefaq.html) & [Seclists](http://seclists.org/fulldisclosure/2013/May/226)  
  **Details here:**  [Slides PHDays 2012 #29](http://fr.slideshare.net/gentilkiwi/mimikatz-phdays/29)

* `sessiondump` by **[Steeve Barbeau](https://twitter.com/steevebarbeau)** @ [HSC](http://www.hsc.fr)  
  [HSC advertising](http://www.hsc.fr/ressources/outils/sessiondump) & [Source](https://github.com/steeve85/metasploit-framework/tree/sessiondump/external/source/meterpreter/source/extensions/sessiondump) & [Pull request](https://github.com/rapid7/metasploit-framework/pull/1750)   
  **Details here:** [core.c](https://github.com/steeve85/metasploit-framework/blob/sessiondump/external/source/meterpreter/source/extensions/sessiondump/core.c)