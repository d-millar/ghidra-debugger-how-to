# STM32F3DISCOVERY

## Context
- OS: macos arm64
- debugger: arm-none-eabi-gdb
- target: STM32F3DISCOVERY

## Set-up for device

- Connect the STM32 board to your computer using the USB cable.
- Install openocd and run:
```
openocd -f /opt/homebrew/share/openocd/scripts/board/stm32f3discovery.cfg 
```

## Terminal Output for Openocd:

```
Open On-Chip Debugger 0.12.0
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
srst_only separate srst_nogate srst_open_drain connect_deassert_srst

Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : clock speed 1000 kHz
Info : STLINK V2J37M26 (API v2) VID:PID 0483:374B
Info : Target voltage: 2.892163
Info : [stm32f3x.cpu] Cortex-M4 r0p1 processor detected
Info : [stm32f3x.cpu] target has 6 breakpoints, 4 watchpoints
Info : starting gdb server for stm32f3x.cpu on 3333
Info : Listening on port 3333 for gdb connections
Info : accepting 'gdb' connection on tcp/3333
```

## Set-up for Ghidra using arm-none-eabi-gdb

- From the Ghidra toolbar, `Configure and launch target_fw.bin using...->remote gdb`.
```
Target: extended-remote
Host: localhost
Port: 3333
Architecture: auto
gdb command: /opt/homebrew/bin/arm-none-eabi-arm
```
- `Launch`

## Set-up for Ghidra using lldb

- From the Ghidra toolbar, `Configure and launch target_fw.bin using...->remote lldb`.
```
Host: localhost
Port: 3333
Architecture: <empty>
gdb command: lldb
```
- `Launch`

## Terminal Output for Ghidra/arm-none-eabi-gdb:

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
Connecting to localhost:3333...Remote debugging using localhost:3333
warning: No executable has been specified and target does not support
determining executable automatically.  Try using the "file" command.
0x080039ba in ?? ()
Connected to Ghidra 11.3 at 127.0.0.1:58926
ARM:LE:32:v8 not found in compiler map
(gdb)
```

## Terminal Output for Ghidra/lldb:

```
(lldb) version
lldb-1600.0.39.109
Apple Swift version 6.0.3 (swiftlang-6.0.3.1.10 clang-1600.0.30.1)
(lldb) script import ghidralldb
(lldb) gdb-remote localhost:3333
(lldb) ghidra trace connect "127.0.0.1:59267"
(lldb) ghidra trace start
Process 1 stopped
* thread #1, stop reason = signal SIGINT
    frame #0: 0x0800299e
->  0x800299e:                                  ; unknown opcode
    0x80029a2:                                  ; unknown opcode
    0x80029a6: ldrlt  r2, [lr, #-0x0]!
    0x80029aa:                                  ; unknown opcode
Target 0: (No executable module.) stopped.
(lldb) ghidra trace sync-enable
(lldb) ghidra trace sync-synth-stopped
add initial events for cli failed
add initial events for target failed
add initial events for process failed
(lldb)
```


## Errors:
- On the first load, there may be a bunch of syntax errors for various patterns, which may be safely ignored.
- As above, `ARM:LE:32:v8` is not explicitly in the compiler map - the default is fine.
- No dynamic memory: use `Regions->Force Full View`
- Static/dynamic listings unsynced: use `Modules->Map the current trace to the current program using identical addresses (<=>)` or equivalent.
