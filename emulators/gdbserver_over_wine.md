# gdbserver via wine

## Context
- OS: macos arm64
- debugger: lldb or gdb
- target: i386 or i686 executables

## Set-up

- Launch the server: 
	- `wine ./gdbserver32.exe localhost:2345 ./Winmine.exe`
- From the Ghidra toolbar, `Configure and launch Winmine.exe using... -> remote lldb` 
 or follow the instructions in `https://github.com/d-millar/ghidra-how-to/blob/main/emulators/using_emulators_with_gdb_on_Apple_Silicon.md`
 
`remote lldb`
```
Host: localhost
Port: 2345
Architecture: i386
lldb command: lldb
```
or 
`gdb via ssh and back` (with `target remote :2345` inlined)
```
Image: 
Arguments:
ssh command: ssh
[User@]Host: 192.168.1.5 (the Linux VM)
Remote Trace RMI Port: 12345
Extra ssh arguments:
gdb command: gdb-multiarch
Run command: starti
Architecture: i386
```
## Terminal Output:
```
(lldb) version
lldb-1600.0.39.3
Apple Swift version 6.0.2 (swiftlang-6.0.2.1.2 clang-1600.0.26.4)
(lldb) script import ghidralldb
(lldb) gdb-remote localhost:2345
(lldb) ghidra trace connect "127.0.0.1:53219"
Process 268 stopped
Target 0: (No executable module.) stopped.
(lldb) ghidra trace start
(lldb) ghidra trace sync-enable
(lldb) ghidra trace sync-synth-stopped
add initial events for cli failed
add initial events for target failed
add initial events for process failed
(lldb) 
```
or
```
GNU gdb (Ubuntu 15.0.50.20240403-0ubuntu1) 15.0.50.20240403-git
Copyright (C) 2024 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "aarch64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
The target architecture is set to "i386".
Remote debugging using :2345
0x7bf96075 in ?? ()
Connected to Ghidra 11.3 at localhost:12345
(gdb) 
```

## Errors:
- For reasons I don't completely understand, ctrl-c (or `interrupt` or `process interrupt`) has no affect for targets running under a wine-emulated gdbserver.  `Kill -INT pid` (suggested in `https://github.com/NationalSecurityAgency/ghidra/issues/6075`) will work, but the failure, for better or worse, is not a function of having Ghidra in the mix.
