# eXDI

## Context
- OS: macos x64
- debugger: windbg over Remote Serial Protocol
- target: Windows kernel

## Set-up

- Recommended reading: **Configuring the EXDI Debugger Transport** (https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/configuring-the-exdi-debugger-transport)
- Also: **Setting Up QEMU Kernel-Mode Debugging Using EXDI** (https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/setting-up-qemu-kernel-mode-debugging-using-exdi)
- The sites above describe how to use the eXDI mode in windbg proper.
- From the Ghidra toolbar, `Configure and launch <anything> using...->dbgeng-kernel`.
```
Python command: py (typically)
Argument: exdi:CLSID={29f9906e-9dbe-4d4b-b0fb-6acf7fb6d014},Kd=Guess,DataBreaks=Exdi
Type: EXDI
Use dbgmodel: either, but dbgmodel is better
Path to dbgeng.dll directory: C:\Program Files (x86)\Windows Kits\10\Debuggers\x64 (typically)
```
- `Launch`
- at the "Launch Failed" timeout dialog, choose "Keep"

## Terminal Output:

```
Connected to Ghidra 11.3 at 127.0.0.1:53328
ConfigFile: C:\Users\llero\Desktop\exdiConfigData.xml
RegMapFile: C:\Users\llero\Desktop\systemregisters.xml

************* Preparing the environment for Debugger Extensions Galler
y repositories **************
   ExtensionRepository : Implicit
   UseExperimentalFeatureForNugetShare : true
   AllowNugetExeUpdate : true
   NonInteractiveNuget : true
   AllowNugetMSCredentialProviderInstall : true
   AllowParallelInitializationOfLocalRepositories : true

   EnableRedirectToV8JsProvider : false

   -- Configuring repositories
      ----> Repository : LocalInstalled, Enabled: true
      ----> Repository : UserExtensions, Enabled: true

>>>>>>>>>>>>> Preparing the environment for Debugger Extensions Galler
y repositories completed, duration 0.015 seconds

************* Waiting for Debugger Extensions Gallery to Initialize **************

>>>>>>>>>>>>> Waiting for Debugger Extensions Gallery to Initialize completed, duration 0.297 seconds
   ----> Repository : UserExtensions, Enabled: true, Packages count: 0   ----> Repository : LocalInstalled, Enable
d: true, Packages count: 29

Microsoft (R) Windows Debugger Version 10.0.26100.1591 AMD64
Copyright (c) Microsoft Corporation. All rights reserved.

Kernel Debugger connection established
Found PE image ntkrnlmp=fffff803`15600000,00000000`01047000">ntkrnlmp @ fffff803`15600000">0xfffff803`15600000    
Found Module Name ntkrnlmp
Found target VersionBlock at 0xfffff80316209998
Connected to Windows 10 22621 x64 target at (Mon Dec 16 14:06:23.735 2024 (UTC - 5:00)), ptr64 TRUE
Symbol search path is: srv*
Executable search path is:
Windows 10 Kernel Version 22621 MP (8 procs) Free x64
Product: WinNt, suite: TerminalServer SingleUserTS
Edition build lab: 22621.1.amd64fre.ni_release.220506-1250
Kernel base = 0xfffff803`15600000 PsLoadedModuleList = 0xfffff803`162134f0
Debug session time: Mon Dec 16 14:04:57.973 2024 (UTC - 5:00)
System Uptime: 0 days 0:44:33.232

This is the Windows Debugger REPL. To drop to Python, type .exit
0: kd> 0: kd> 
```

If launched with dbgmodel checked, the Model View should contain per-process/thread information in the primary 
**Processes** sub-tree.  If launched with dbgmodel off, the **Processes** tree will contain one entry at 0xf0f0f0f0 with
one "thread" entry per processor.  The target process/thread list will reside under the **ExdiProcesses** node.

## Notes:
	Setup for EXDI connections is fairly complicated and difficult to get correct. The argument string typically 
	should be something like:
```
 exdi:CLSID={29f9906e-9dbe-4d4b-b0fb-6acf7fb6d014},Kd=Guess,DataBreaks=Exdi
``` 
	The CLSID here should match the CLSID in the **exdiConfigData.xml** file in the debugger architectural directory.  If windbg has been
    run using EXDI at some point, there will also be an entry in the System Registry for this CLSID.  The InprocServer32 subentry
    for this CLSID in the Registry should point to a copy of ExdiGdbSrv.dll, typically the one in the same directory. This DLL
    must reside somewhere that the debugger has permission to load from, i.e.  not in the WindowsApps directory tree.
    The **exdiConfigData** file should be configured for the target you're using. I'd heavily recommend using **displayCommPackets==yes**,
    as many of the tasks take considerable time, and this is the only indicator of progress.

    The **Kd=Guess* parameter causes the underlying engine to scan memory for the kernel's base address, which will probably not
    be provided by the gdbstub. **Kd=NtBaseAddr** is also a valid option, as is eliminating the parameter, but, currently, I have no
    idea how to point the configuration at a correct value. Using this option will cause the load to spin pointlessly.)  If you can,
    I highly recommend breaking the target near the base address (e.g. by using the windbg in non-EXDI mode simultaneously), 
	as the search proceeds down through memory starting at the current program counter.  If the difference between the PC and
	the base address is large, the loading process will punt before useful values are detected. (If anyone understand how to extend 
	this search (or knows how to set the base address to sidestep the scan), I would really love some guidance.)  Also,
	bear in mind, if the target is booted in non-debug mode, the search process is guaranteed to fail with the resulting
	reduction in available commands.
