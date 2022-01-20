# Lab 8: Forensics

## **Memory Forensics**

This is adapted from [https://github.com/stuxnet999/MemLabs/blob/master/Lab%200/README.md](https://github.com/stuxnet999/MemLabs/blob/master/Lab%200/README.md)

Memory forensics involves the analysis of a snapshot of memory in a computer that preserves the running state of the computer at the time the snapshot was taken. This allows foresenic examiners to see what users may have been doing on the computer at the time, what processes were running and what data they may have been accessing.&#x20;

We can use memory forensics to identify malware that may have been only resident in memory and never saved on disk. We can also use it for evidence as to the activities of a user at a given time.&#x20;

## Taking a Memory Snapshot

There are a variety of software programs that will take a memory snapshot suitable for analysis:

* Windows
  * Dumpit
  * FTK Imager
  * Acquire
* Linux
  * AVML
  * LiME (also does Android)
* Mac
  * MacQuisition

These tools create a file that can then be analysed using tools such as Volatility that we are going to use in this lab.

### **Memory Analysis**

We'll be analyzing the memory dump file (**Challenge.raw**) using **Volatility 2.6.**

Volatility 2 needs to know the version of operating system that the memory dump was from and this determines the specific profile that Volatility uses. To determine this, we can use the command imageinfo.

To get started, run the docker container:

{% tabs %}
{% tab title="Windows/Apple Intel" %}
```bash
docker pull cybernemosyne/cits1003:volatility
docker run -it cybernemosyne/cits1003:volatility
```
{% endtab %}

{% tab title="Apple Silicon" %}
```
docker pull cybernemosyne/cits1003:volatility-x
docker run -it cybernemosyne/cits1003:volatility-x
```
{% endtab %}
{% endtabs %}

Change into the directory /opt/memory

Run the command **imageinfo** to get the following output:

```bash
root@3fc5f73f4a8e:/opt/memory# volatility -f Challenge.raw imageinfo
Volatility Foundation Volatility Framework 2.6.1
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win7SP1x86_23418, Win7SP0x86, Win7SP1x86_24000, Win7SP1x86
                     AS Layer1 : IA32PagedMemoryPae (Kernel AS)
                     AS Layer2 : FileAddressSpace (/opt/memory/Challenge.raw)
                      PAE type : PAE
                           DTB : 0x185000L
                          KDBG : 0x8273cb78L
          Number of Processors : 1
     Image Type (Service Pack) : 1
                KPCR for CPU 0 : 0x80b96000L
             KUSER_SHARED_DATA : 0xffdf0000L
           Image date and time : 2018-10-23 08:30:51 UTC+0000
     Image local date and time : 2018-10-23 14:00:51 +0530
root@3fc5f73f4a8e:/opt/memory# 
```

The output returns a number of possible profiles that could be used. We can refine this by using another plugin called **kdbgscan**

```bash
root@3fc5f73f4a8e:/opt/memory# volatility -f Challenge.raw kdbgscan 
Volatility Foundation Volatility Framework 2.6.1
**************************************************
Instantiating KDBG using: /opt/memory/Challenge.raw WinXPSP2x86 (5.1.0 32bit)
Offset (P)                    : 0x273cb78
KDBG owner tag check          : True
Profile suggestion (KDBGHeader): Win7SP1x86_23418
Version64                     : 0x273cb50 (Major: 15, Minor: 7601)
PsActiveProcessHead           : 0x82751d70
PsLoadedModuleList            : 0x82759730
KernelBase                    : 0x82604000

**************************************************
Instantiating KDBG using: /opt/memory/Challenge.raw WinXPSP2x86 (5.1.0 32bit)
Offset (P)                    : 0x273cb78
KDBG owner tag check          : True
Profile suggestion (KDBGHeader): Win7SP1x86_24000
Version64                     : 0x273cb50 (Major: 15, Minor: 7601)
PsActiveProcessHead           : 0x82751d70
PsLoadedModuleList            : 0x82759730
KernelBase                    : 0x82604000

**************************************************
Instantiating KDBG using: /opt/memory/Challenge.raw WinXPSP2x86 (5.1.0 32bit)
Offset (P)                    : 0x273cb78
KDBG owner tag check          : True
Profile suggestion (KDBGHeader): Win7SP1x86
Version64                     : 0x273cb50 (Major: 15, Minor: 7601)
PsActiveProcessHead           : 0x82751d70
PsLoadedModuleList            : 0x82759730
KernelBase                    : 0x82604000

**************************************************
Instantiating KDBG using: /opt/memory/Challenge.raw WinXPSP2x86 (5.1.0 32bit)
Offset (P)                    : 0x273cb78
KDBG owner tag check          : True
Profile suggestion (KDBGHeader): Win7SP0x86
Version64                     : 0x273cb50 (Major: 15, Minor: 7601)
PsActiveProcessHead           : 0x82751d70
PsLoadedModuleList            : 0x82759730
KernelBase                    : 0x82604000

```

So we can now use the profile WinXPSP2x86 which is the Windows XP Service Pack 2 version of the OS.

Now as a forensic analyst, one of the most important things we would like to know from a system during analysis would be:

* Active processes
* Commands executed in the shell/terminal/Command prompt
* Hidden processes (if any) or Exited processes
* Browser History (This is very much subjective to the scenario involved)

And many more...

Now, to list the active or running processes, we use the help of the plugin **pslist**.

```bash
root@3fc5f73f4a8e:/opt/memory# volatility -f Challenge.raw --profile=Win7SP1x86 pslist
Volatility Foundation Volatility Framework 2.6.1
Offset(V)  Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit                          
---------- -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0x83d09c58 System                    4      0     85      483 ------      0 2018-10-23 08:29:16 UTC+0000                                 
0x8437db18 smss.exe                260      4      2       29 ------      0 2018-10-23 08:29:16 UTC+0000                                 
0x84d69030 csrss.exe               340    332      8      347      0      0 2018-10-23 08:29:21 UTC+0000                                 
0x84d8d030 csrss.exe               380    372      9      188      1      0 2018-10-23 08:29:23 UTC+0000                                 
0x84d93c68 wininit.exe             388    332      3       79      0      0 2018-10-23 08:29:23 UTC+0000                                 
0x84dcbd20 winlogon.exe            424    372      6      117      1      0 2018-10-23 08:29:23 UTC+0000                                 
0x84debd20 services.exe            484    388     10      191      0      0 2018-10-23 08:29:25 UTC+0000                                 
0x84def3d8 lsass.exe               492    388      7      480      0      0 2018-10-23 08:29:25 UTC+0000                                 
0x84df2378 lsm.exe                 500    388     10      146      0      0 2018-10-23 08:29:25 UTC+0000                                 
0x84e23030 svchost.exe             592    484     12      358      0      0 2018-10-23 08:29:30 UTC+0000                                 
0x84e41708 VBoxService.ex          652    484     12      116      0      0 2018-10-23 08:29:31 UTC+0000                                 
0x84e54030 svchost.exe             716    484      9      243      0      0 2018-10-23 08:29:32 UTC+0000                                 
0x84e7ad20 svchost.exe             804    484     19      378      0      0 2018-10-23 08:29:32 UTC+0000                                 
0x84e84898 svchost.exe             848    484     20      400      0      0 2018-10-23 08:29:33 UTC+0000                                 
0x84e89c68 svchost.exe             872    484     19      342      0      0 2018-10-23 08:29:33 UTC+0000                                 
0x84e8c648 svchost.exe             896    484     30      809      0      0 2018-10-23 08:29:33 UTC+0000                                 
0x84ea7d20 audiodg.exe             988    804      6      127      0      0 2018-10-23 08:29:35 UTC+0000                                 
0x84f033c8 svchost.exe            1192    484     15      365      0      0 2018-10-23 08:29:40 UTC+0000                                 
0x84f323f8 spoolsv.exe            1336    484     16      295      0      0 2018-10-23 08:29:43 UTC+0000                                 
0x84f4dca0 svchost.exe            1364    484     19      307      0      0 2018-10-23 08:29:43 UTC+0000                                 
0x84f7d578 svchost.exe            1460    484     11      148      0      0 2018-10-23 08:29:44 UTC+0000                                 
0x84f828f8 svchost.exe            1488    484      8      170      0      0 2018-10-23 08:29:44 UTC+0000                                 
0x850b2538 taskhost.exe            308    484      8      151      1      0 2018-10-23 08:29:55 UTC+0000                                 
0x850d0030 sppsvc.exe             1164    484      6      154      0      0 2018-10-23 08:29:57 UTC+0000                                 
0x85109030 dwm.exe                1992    848      5      132      1      0 2018-10-23 08:30:04 UTC+0000                                 
0x85097870 explorer.exe            324   1876     33      827      1      0 2018-10-23 08:30:04 UTC+0000                                 
0x85135af8 VBoxTray.exe           1000    324     14      159      1      0 2018-10-23 08:30:08 UTC+0000                                 
0x85164030 SearchIndexer.         2032    484     14      614      0      0 2018-10-23 08:30:14 UTC+0000                                 
0x8515ad20 SearchProtocol          284   2032      7      235      0      0 2018-10-23 08:30:16 UTC+0000                                 
0x8515cd20 SearchFilterHo         1292   2032      5       80      0      0 2018-10-23 08:30:17 UTC+0000                                 
0x851a6610 cmd.exe                2096    324      1       22      1      0 2018-10-23 08:30:18 UTC+0000                                 
0x851a5cd8 conhost.exe            2104    380      2       52      1      0 2018-10-23 08:30:18 UTC+0000                                 
0x845a8d20 DumpIt.exe             2412    324      2       38      1      0 2018-10-23 08:30:48 UTC+0000                                 
0x84d83d20 conhost.exe            2424    380      2       51      1      0 2018-10-23 08:30:48 UTC+0000                                 
root@3fc5f73f4a8e:/opt/memory# 
```

Executing this command gives us a list of processes which were running when the memory dump was taken. The output of the command gives a fully formatted view which includes the name, PID, PPID, Threads, Handles, start time etc..

Observing closely, we notice some processes which require some attention.

* **cmd.exe**
  * This is the process responsible for the command prompt. Extracting the content from this process might give us the details as to what commands were executed in the system
* **DumpIt.exe**
  * This process was used by me to acquire the memory dump of the system.
* **Explorer.exe**
  * This process is the one which handles the File Explorer.

Now since we have seen that **cmd.exe** was running, let us try to see if there were any commands executed in the shell/terminal.

For this, we use the **cmdscan** plugin.

```bash
root@3fc5f73f4a8e:/opt/memory# volatility -f Challenge.raw --profile=Win7SP1x86 cmdscan
Volatility Foundation Volatility Framework 2.6.1
**************************************************
CommandProcess: conhost.exe Pid: 2104
CommandHistory: 0x300498 Application: cmd.exe Flags: Allocated, Reset
CommandCount: 1 LastAdded: 0 LastDisplayed: 0
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x5c
Cmd #0 @ 0x2f43c0: C:\Python27\python.exe C:\Users\hello\Desktop\demon.py.txt
Cmd #12 @ 0x2d0039: ???
Cmd #19 @ 0x300030: ???
Cmd #22 @ 0xff818488: ?
Cmd #25 @ 0xff818488: ?
Cmd #36 @ 0x2d00c4: /?0?-???-
Cmd #37 @ 0x2fd058: 0?-????
**************************************************
CommandProcess: conhost.exe Pid: 2424
CommandHistory: 0x2b04c8 Application: DumpIt.exe Flags: Allocated
CommandCount: 0 LastAdded: -1 LastDisplayed: -1
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x5c
Cmd #22 @ 0xff818488: ?
Cmd #25 @ 0xff818488: ?
Cmd #36 @ 0x2800c4: *?+?(???(
Cmd #37 @ 0x2ad070: +?(????
```



If you can see from the above, a Python file was executed. The executed command was

&#x20;`C:\Python27\python.exe C:\Users\hello\Desktop\demon.py.txt`

So our next step would be check if this python script sent any output to **stdout**. For this, we use the **consoles** plugin.

```bash
root@3fc5f73f4a8e:/opt/memory# volatility -f Challenge.raw --profile=Win7SP1x86 consoles
Volatility Foundation Volatility Framework 2.6.1
**************************************************
ConsoleProcess: conhost.exe Pid: 2104
Console: 0xe981c0 CommandHistorySize: 50
HistoryBufferCount: 2 HistoryBufferMax: 4
OriginalTitle: %SystemRoot%\system32\cmd.exe
Title: C:\Windows\system32\cmd.exe
AttachedProcess: cmd.exe Pid: 2096 Handle: 0x5c
----
CommandHistory: 0x300690 Application: python.exe Flags: 
CommandCount: 0 LastAdded: -1 LastDisplayed: -1
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x0
----
CommandHistory: 0x300498 Application: cmd.exe Flags: Allocated, Reset
CommandCount: 1 LastAdded: 0 LastDisplayed: 0
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x5c
Cmd #0 at 0x2f43c0: C:\Python27\python.exe C:\Users\hello\Desktop\demon.py.txt
----
Screen 0x2e6368 X:80 Y:300
Dump:
Microsoft Windows [Version 6.1.7601]                                            
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.                 
                                                                                
C:\Users\hello>C:\Python27\python.exe C:\Users\hello\Desktop\demon.py.txt       
335d366f5d6031767631707f  
<SNIP...>
```

We see that a certain string `335d366f5d6031767631707f` has been written out to **stdout**. If you put this text into a HEX to ASCII converter online, you will see that it produces rubbish and so it is likely to be encrypted.&#x20;

We will leave this for now and carry on with our investigation.

All operating systems support the concept of environment variables. These can be used by programs to customise how they are run, where they look for, or write, files, etc. We can use the envars command for volatility:

```bash
root@3fc5f73f4a8e:/opt/memory# volatility -f Challenge.raw --profile=Win7SP1x86 envars  
Volatility Foundation Volatility Framework 2.6.1
Pid      Process              Block      Variable                       Value
-------- -------------------- ---------- ------------------------------ -----
<SNIP...>
    2096 cmd.exe              0x00384be0 ALLUSERSPROFILE                C:\ProgramData
    2096 cmd.exe              0x00384be0 APPDATA                        C:\Users\hello\AppData\Roaming
    2096 cmd.exe              0x00384be0 CommonProgramFiles             C:\Program Files\Common Files
    2096 cmd.exe              0x00384be0 COMPUTERNAME                   HELLO-PC
    2096 cmd.exe              0x00384be0 ComSpec                        C:\Windows\system32\cmd.exe
    2096 cmd.exe              0x00384be0 FP_NO_HOST_CHECK               NO
    2096 cmd.exe              0x00384be0 HOMEDRIVE                      C:
    2096 cmd.exe              0x00384be0 HOMEPATH                       \Users\hello
    2096 cmd.exe              0x00384be0 LOCALAPPDATA                   C:\Users\hello\AppData\Local
    2096 cmd.exe              0x00384be0 LOGONSERVER                    \\HELLO-PC
    2096 cmd.exe              0x00384be0 NUMBER_OF_PROCESSORS           1
    2096 cmd.exe              0x00384be0 OS                             Windows_NT
    2096 cmd.exe              0x00384be0 Path                           C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\
    2096 cmd.exe              0x00384be0 PATHEXT                        .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
    2096 cmd.exe              0x00384be0 PROCESSOR_ARCHITECTURE         x86
    2096 cmd.exe              0x00384be0 PROCESSOR_IDENTIFIER           x86 Family 6 Model 142 Stepping 9, GenuineIntel
    2096 cmd.exe              0x00384be0 PROCESSOR_LEVEL                6
    2096 cmd.exe              0x00384be0 PROCESSOR_REVISION             8e09
    2096 cmd.exe              0x00384be0 ProgramData                    C:\ProgramData
    2096 cmd.exe              0x00384be0 ProgramFiles                   C:\Program Files
    2096 cmd.exe              0x00384be0 PROMPT                         $P$G
    2096 cmd.exe              0x00384be0 PSModulePath                   C:\Windows\system32\WindowsPowerShell\v1.0\Modules\
    2096 cmd.exe              0x00384be0 PUBLIC                         C:\Users\Public
    2096 cmd.exe              0x00384be0 SESSIONNAME                    Console
    2096 cmd.exe              0x00384be0 SystemDrive                    C:
    2096 cmd.exe              0x00384be0 SystemRoot                     C:\Windows
    2096 cmd.exe              0x00384be0 TEMP                           C:\Users\hello\AppData\Local\Temp
    2096 cmd.exe              0x00384be0 Thanos                         xor and password
    2096 cmd.exe              0x00384be0 TMP                            C:\Users\hello\AppData\Local\Temp
    2096 cmd.exe              0x00384be0 USERDOMAIN                     hello-PC
    2096 cmd.exe              0x00384be0 USERNAME                       hello
    2096 cmd.exe              0x00384be0 USERPROFILE                    C:\Users\hello
    2096 cmd.exe              0x00384be0 windir                         C:\Windows
    2096 cmd.exe              0x00384be0 windows_tracing_flags          3
    2096 cmd.exe              0x00384be0 windows_tracing_logfile        C:\BVTBin\Tests\installpackage\csilogfile.log
    <SNIP...>
```

We are mainly interested in the environment variables for the cmd.exe process that was running the Python file. If you look at the environment variables, there is one called Thanos (A fictional supervillain) that has text which says "xor and password". So, let us try and brutefoce the hex characters we got with XOR.&#x20;

We can do this in Cyberchef using 2 recipes. The first is FromHex which converts the Hex characters to ASCII, the second is XOR Brute Force which will successively try numbers from 1 to 100  and XOR them against the input. Looking through the output, we see that the 2nd entry is "1\_4m\_b3tt3r}" which looks like slightly meaningful text since it is Leet for "I am better" (and has the format of half of a flag for the challenge).&#x20;

Volatility can also extract password hashes from accounts and we can try that using the **hashdump** command.

Well, the next part is `password`. Using volatility, we can extract the NTLM password hashes using the **hashdump** plugin.

```bash
root@3fc5f73f4a8e:/opt/memory# volatility -f Challenge.raw --profile=Win7SP1x86 hashdump
Volatility Foundation Volatility Framework 2.6.1
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
hello:1000:aad3b435b51404eeaad3b435b51404ee:101da33f44e92c27835e64322d72e8b7:::
```

These hashes are in a particular format that Microsoft used for passwords called NTLM. The specific hash we are interested in is the NT hash which is the number : `101da33f44e92c27835e64322d72e8b7`.&#x20;

To crack this, we are going to use a program called John The Ripper, or john for short. Create a file called hash.txt and insert the entire hash line into it:

```bash
hello:1000:aad3b435b51404eeaad3b435b51404ee:101da33f44e92c27835e64322d72e8b7:::
```

Then run john using the following command:

```bash
..:/opt/memory# /opt/john/run/john  --pot=john.pot --format=NT --wordlist=passwords.txt hash.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
Warning: Only 13 candidates left, minimum 24 needed for performance.
flag{you_are_good_but (hello)     
1g 0:00:00:00 DONE (2021-07-06 05:34) 100.0g/s 1300p/s 1300c/s 1300C/s 123456..1234567890
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed. 
```

In this command, we passed a passwords.txt file that contains the passwords we are going to try and brute force using John. This example is slightly contrived because I have added the specific one that gives us the password we are looking for but John works very well normally because people tend to use  very easy to guess passwords.&#x20;

Once we run John, we get a hit and the passwords is confirmed to be the other half of the flag --> **flag{you\_are\_good\_but**. Concatenating the 2 parts gives us the whole flag.

### Question 1. What was the whole flag?

FLAG: Enter the flag to claim the prize

