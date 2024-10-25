# mGBA

## Context
- OS: macos arm64
- debugger: lldb
- target: mGBA

## Set-up

- Recommended reading: [Link] (https://wrongbaud.github.io/posts/ghidra-debugger/ A first look at Ghidra's Debugger - Game Boy Advance Edition)
- Also: [Link] (https://github.com/SiD3W4y/GhidraGBA GhidraGBA)
- The sites above describe how to install a GBA loader extension for Ghidra and use it to analyze a target ROM.
- To launch your target: `mgba --gdb /Path/to/target.gba` (should launch stopped/waiting for a connection).
- From the Ghidra toolbar, "Configure and launch target.gba using..."->"remote lldb".
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
- Static/dynamic listings unsynced: use "Modules"->"Map the current trace to the current program using identical addresses" (<=>) or equivalent.
