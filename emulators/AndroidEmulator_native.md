# Android (native)

## Context
- OS: macos arm64
- debugger: lldb
- target: AndroidStudio (emulator) native targets

## Set-up

- To connect lldb: 
	- cd `/Applications/Android\ Studio.app/Contents/plugins/android-ndk/resources/lldb/android`
	- `adb push ./arm64-v8a/lldb-server /data/local/tmp`
	- `adb shell` -> `cd /data/local/tmp`
	- `./lldb-server platform --listen "*:54321" --server`
	- (from a new terminal window) `adb forward tcp:54321 tcp:54321`
	- `adb devices`  (I get `emulator-5554 device`)
- To test using lldb (without Ghidra): 
	- `lldb`
	- `platform select remote-android` (gives the "Connected: no" message)
	- `platform connect connect://emulator-5554:54321`
	- `file target`; `b main`; `r`
- From the Ghidra toolbar, `Configure and launch target using... -> android gdb` 
```
Image: /data/local/tmp/target (this probably will require at least a path modification)
Argument: 
Host: emulator-5554
Port: 54321
Architecture:
lldb command: lldb
Run command: process launch --stop-at-entry
```

## Terminal Output:

```
(lldb) version
lldb-1600.0.39.3
Apple Swift version 6.0.2 (swiftlang-6.0.2.1.2 clang-1600.0.26.4)
(lldb) script import ghidralldb
(lldb) platform select remote-android
  Platform: remote-android
 Connected: no
(lldb) platform connect connect://emulator-5554:54321
  Platform: remote-android
    Triple: aarch64-unknown-linux-android
OS Version: 30 (5.4.249-android11-2-00004-g212a7aaded26-ab10688362)
  Hostname: localhost
 Connected: yes
WorkingDir: /data/local/tmp
    Kernel: #1 SMP PREEMPT Mon Aug 21 12:17:45 UTC 2023
(lldb) target create "/data/local/tmp/target"
warning: (aarch64) probably a lot of warnings 
Current executable set to '/data/local/tmp/target' (aarch64).
(lldb) ghidra trace connect "127.0.0.1:59586"
(lldb) ghidra trace start
(lldb) ghidra trace sync-enable
(lldb) ghidra trace sync-synth-stopped
add listener for process failed
add initial events for cli failed
add initial events for target failed
add initial events for process failed
Couldn't record page with PC: Cannot convert '$pc' to address: error: 
:1: use of undeclared identifier '$pc'
    1 | $pc
      | ^

Couldn't record page with SP: Cannot convert '$sp' to address: error: 
:1: use of undeclared identifier '$sp'
    1 | $sp
      | ^

Exception in batch operation: TraceRmiError('ghidra.app.plugin.core.de
TraceRmiHandler.InvalidObjPathError')
(lldb) process launch --stop-at-entry
Process 6568 launched: '/Users/llero/.lldb/module_cache/remote-android
-9D98-4082-3E5AB7448BAC/target' (aarch64)
Process 6568 stopped
* thread #1, name = 'target', stop reason = signal SIGSTOP
    frame #0: 0x0000007ff7f53bb0 linker64`__dl__start
linker64`__dl__start:
->  0x7ff7f53bb0 <+0>: mov    x0, sp
    0x7ff7f53bb4 <+4>: bl     0x7ff7f86bd4   ; __dl___linker_init
    0x7ff7f53bb8 <+8>: br     x0
    0x7ff7f53bbc:      udf    #0x0
Target 0: (toybox) stopped.
(lldb)
```

## Errors:
- The start-up errors should probably be suppressed, but currently they do not repeat on step, so....
