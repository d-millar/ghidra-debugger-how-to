# mGBA

## Context
- OS: Ubuntu24 arm64
- debugger: mips64-gdb
- target: Ares emulator

## Set-up no

- Discussion: **Problems Getting The Debugger To Work With Nintendo 64** (https://github.com/NationalSecurityAgency/ghidra/discussions/6787)
- To launch your target: `./ares` then load `target.v64`.
- From the Ghidra toolbar, `Configure and launch target.v64 using...->remote gdb`.
```
Host: localhost
Port: 9123
gdb command: /opt/n64/mips64-gdb
Architecture: mips:4000
Endian: auto
```
- `Launch`

## Terminal Output:

```
GNU gdb (GDB) 16.1
Copyright (C) 2024 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "--host=aarch64-unknown-linux-gnu --target=mips64".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
The target architecture is set to "mips:4000".
The target endianness is set automatically (currently big endian).
Connecting to locahost:9123...Remote debugging using localhost:9123
Warning: No executable has been specified and target does not support
determining executable automatically.  Try using the "file" command.
0x800000170 in ?? ()
Connected to Ghidra 11.3 at 127.0.0.1:39751
MIPS:BE:32:default not found in compiler map - using default compiler
(gdb)
```

## Errors:
- You may get a stack trace and `gdb.error: Cannot execute this command while the target is running`
 if the target is running. Hit break if so.
- You may get `GDB is unable to find the start of the function` and related errors if you haven't loaded a target
 or if the target is not mapped correctly.
- Static/dynamic listings unsynced: use `Modules->Map the current trace to the current program using identical addresses (<=>)` or equivalent.


