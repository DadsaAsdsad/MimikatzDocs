This is the _main_ module of `mimikatz`, it contains quick commands to operate with the tool.  
For this particular one, no need to prefix command by the module name (but it works too), eg: `exit` is the same as `standard::exit`.

> Commands: [exit](#exit), [cls](#cls), [answer](#answer), [coffe](#coffee), [sleep](#sleep), [log](#log), [base64](#base64), [version](#version), [cd](#cd)

## exit
Quits `mimikatz`, after cleaning routines.
```
mimikatz # exit
Bye!
```

## cls
Clears screen, by filling the console window with spaces.
```
mimikatz # cls
```
**Remark:** it does not work with remote execution tools like `psexec`, `meterpreter` or others.

## answer
Gives the Answer to the Ultimate Question of Life, the Universe, and Everything.
```
mimikatz # answer
42.
```

## coffee
Because everyone deserves a good coffee.
```
mimikatz # coffee

    ( (
     ) )
  .______.
  |      |]
  \      /
   `----'
```

## sleep
Sleeps an amount of milliseconds (1000 ms by default).

**Argument:**
* `number` - _optional_ - the number of milliseconds to sleep (default is 1000)
```
mimikatz # sleep
Sleep : 1000 ms... End !

mimikatz # sleep 4200
Sleep : 4200 ms... End !
```

## log
Logs all outputs to a file (`mimikatz.log` by default).

**Arguments:**
* `filename` - _optional_ - the file name for the log file
* `/stop` - _optional_ - stop the file logging
```
mimikatz # log
Using 'mimikatz.log' for logfile : OK

mimikatz # log other.log
Using 'other.log' for logfile : OK

mimikatz # log /stop
Using '(null)' for logfile : OK
```

## base64
Switches from file writing on the disk, to `Base64` output instead.
```
mimikatz # base64
isBase64Intercept was    : false
isBase64Intercept is now : true

mimikatz # kerberos::list /export

[00000000] - 17
   Start/End/MaxRenew: 24/04/2014 08:24:20 ; 24/04/2014 18:17:29 ; 01/05/2014 08:17:29
   Server Name       : krbtgt/CHOCOLATE.LOCAL @ CHOCOLATE.LOCAL
   Client Name       : Administrateur @ CHOCOLATE.LOCAL
   Flags 60a00000    : pre_authent ; renewable ; forwarded ; forwardable ;
====================
Base64 of file : 0-60a00000-Administrateur@krbtgt~CHOCOLATE.LOCAL-CHOCOLATE.LOCAL.kirbi
====================
doIFOTCCBTWgAwIBBaEDAgEWooIELjCCBCphggQmMIIEIqADAgEFoREbD0NIT0NP
TEFURS5MT0NBTKIkMCKgAwIBAqEbMBkbBmtyYnRndBsPQ0hPQ09MQVRFLkxPQ0FM
...
GA8yMDE0MDQyNDIyNTE0NFqnERgPMjAxNDA1MDExMjUxNDRaqBEbD0NIT0NPTEFU
RS5MT0NBTKkkMCKgAwIBAqEbMBkbBmtyYnRndBsPQ0hPQ09MQVRFLkxPQ0FM
====================

   * Saved to file     : 0-60a00000-Administrateur@krbtgt~CHOCOLATE.LOCAL-CHOCOLATE.LOCAL.kirbi
```
**Remark:** Commands that want to write file on disk think they do (they indicate that files are saved to disk)

## version
Displays versions of `mimikatz` and `Windows`
```
mimikatz # version

mimikatz 2.0 alpha (arch x86)
Windows NT 6.1 build 7601 (arch x64)
msvc 150030729 207
```

* `150030729 207` is the WinDDK compiler
* `150030729 1` is the vc90 compiler (Visual Studio 2008)
* `160040219 1` is the vc100 compiler (Visual Studio 2010)
* `170061030 0` is the vc110(_xp) compiler (Visual Studio 2012)
* `180040629 0` is the vc120(_xp) compiler (Visual Studio 2013)
* `190023026 0` is the vc140(_xp) compiler (Visual Studio 2015)


## cd
Change or display current directory

**Argument:**
* `directory` - _optional_ - the directory you want to go
```
mimikatz # cd
C:\security\mimikatz\mimikatz

mimikatz # cd x:\vm
Old: C:\security\mimikatz\mimikatz
New: x:\vm
```