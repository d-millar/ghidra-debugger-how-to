# PCSX-Redux

## Context
- OS: macos arm64
- debugger: gdb-multiarch
- target: PCSX-Redux

## Set-up

- Recommended reading: **Connecting PCSX-Redux to Ghidra** (https://pcsx-redux.consoledev.net/Debugging/ghidra)
- Also: **Debugger not working for bare mips target. #3169** (https://github.com/NationalSecurityAgency/ghidra/issues/3169)
- The first site provides the developer's guidance on running the emulator; the second, some historical issues.
- To launch your target: 
	- Configure the PCSX-Redux with `Emulation Configuration -> Enable Debugger & Enable GDB Server`
	- Load your target game: `File -> Open Disk Image...`
- From the Ghidra toolbar, `Configure and launch openbios.bin|target.cue using... -> remote gdb` 
```
Host: localhost
Port: 3333
Architecture: mips:3000
lldb command: gdb-multiarch 
```
(or follow the instructions in `using_emulators_with_gdb_on_Apple_Silicon.md`)
- `Launch`
- From the Terminal, `stepi` to get the target to break.
- From Regions, use `Force Full View` to populate the Dynamic Listing.
- From Static Mappings, use `Add Mappings from Listing Selections` to coordinate the Static and Dynamic Listing views.

## Terminal Output:

```
The target architecture is set to "mips:3000"
GNU gdb (standard header not repeated here)
Remote debugging using :3333
Warning: No executable has been specified and the target does not support determining executable automatically. Try using the "file" command.
0x80062f3c in ?? ()
Connected to Ghidra 11.3 at localhost:12345
(gdb) stepi
warning: GDB can't find the start of the function at 0x80062f3c.
(extended warning skipped))

```

## Errors:
- You'll probably see the "can't find the start of the function" warning twice on every step/break.  Not sure how to suppress this at present.
- If you see sign-extended addresses, particularly in the Dynamic Listing, Ghidra has failed to identify your language spec. This can be fixed by either modifying the tables in ghidragdb/arch.py and re-compiling (easier for source than distributions) or forcing the target language: 
	- `Debugger->Choose Platform->More...`
	- uncheck `Show Only Recommended Offers`
	- select your best match
