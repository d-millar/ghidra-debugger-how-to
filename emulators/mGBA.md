# mGBA

## Context
- OS: macos arm64
- debugger: lldb / arm-none-eabi-gdb
- target: mGBA

## Set-up

- Recommended reading: **A first look at Ghidra's Debugger - Game Boy Advance Edition** (https://wrongbaud.github.io/posts/ghidra-debugger)
- Also: **GhidraGBA** (https://github.com/SiD3W4y/GhidraGBA)
- The sites above describe how to install a GBA loader extension for Ghidra and use it to analyze a target ROM.
- To launch your target: `sdl/mgba --gdb /Path/to/target.gba` (should launch stopped/waiting for a connection).
- From the Ghidra toolbar, `Configure and launch target.gba using...->remote lldb`.
```
Host: localhost
Port: 2345
Architecture: [empty]
lldb command: lldb 
```
- `Launch`

## Terminal Output:

```
(lldb) version
lldb-1600.0.36.3
Apple Swift version 6.0 (swiftlang-6.0.0.9.10 clang-1600.0.26.2)
(lldb) script import ghidralldb
(lldb) gdb-remote localhost:2345
(lldb) ghidra trace connect "127.0.0.1:57689"
Process 1 stopped
* thread #1, stop reason = signal SIGINT
    frame #0: 0x00000000
->  0x0: andeq r0, r0, r0
    0x0: andeq r0, r0, r0
    0x0: andeq r0, r0, r0
target 0: (No executable module.) stopped.
(lldb) ghidra trace start
(lldb) ghidra trace sync-enable
(lldb) ghidra trace sync-synth-stopped
add initial events for cli failed
add initial events for target failed
add initial events for process failed
error: Assertion failed:....
```

## Errors:
- If you're seeing the "Assertion failed" error, there's no current workaround. It has no affect on execution (other than flooding the terminal window in an annoying way on every step).
- Static/dynamic listings unsynced: use `Modules->Map the current trace to the current program using identical addresses (<=>)` or equivalent.


## Set-up

- Launch your target as above
- From the Ghidra toolbar, `Configure and launch target.gba using...->remote gdb`.
```
Target: remote
Host: localhost
Port: 2345
Architecture: auto
gdb command: arm-none-eabi-gdb
```
- `Launch`

```
GNU gdb (GDB) 15.2
Copyright (C) 2024 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "--host=aarch64-apple-darwin24.0.0 --target=arm-none-eabi".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
The target architecture is set to "auto" (currently "arm").
Connecting to localhost:2345...Remote debugging using localhost:2345
warning: No executable has been specified and target does not support
determining executable automatically.  Try using the "file" command.
0x00000000 in ?? ()
Connected to Ghidra 11.3 at 127.0.0.1:58453
(gdb) 
```
