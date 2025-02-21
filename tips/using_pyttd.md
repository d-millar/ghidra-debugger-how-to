# using pyttd

## Context
- OS: Windows
- debugger: ghidrattd
- target: TTD trace

## Set-up

- Recommended reading: **Time Travel Debugging Overview** (https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/time-travel-debugging-overview)
- Prerequisites: **ttd-bindings** (https://github.com/commial/ttd-bindings) and **pybind11** (https://github.com/pybind/pybind11)

### Getting started
- Download or build and install `pybind`.
- Download or build the latest `pyttd.pyd`, making sure the version of Python you intend to use is reflected in the `TTD.sln`.
- Copy `TTDReplay.dll` and `TTDReplayCPU.dll` from Windbg2 `ttd` directory, Window Kits `x64\ttd` directory, or some preferably un-system-protected equivalent into the `site-packages\pyttd` directory for the matching version of Python.  (Should have an empty `__init__.py`, `pyttd.pyd`, and the two TTD DLLs.

### Starting the debugger

- From the Ghidra toolbar, `Configure and launch target.exe using...->ttd`.
```
Python command: python3 (or full-path to matching version)
Trace (.run): target.run
Arguments: 
Use dbgmodel: (checked)
Path to dbgeng.dll & \ttd: C:\Program Files\Amazon Corretto\jdk21.0.3_9\bin
  (or wherever `ttd\TTDReplay.dll` was copied from)
```
- `Launch`

## Terminal Output:

```
Loading dbgeng and friends from C:\Program Files\Amazon Corretto\jdk21.0.3_9\bin
Connected to Ghidra 11.4 at 127.0.0.1:60054
Trace from <Position 2d:0> to <Position 1be15:5bf>
calling start with C:\Users\dbmilla\Documents\notepad01.run
started
[2:0] Module C:\Windows\target.exe loaded
[3:0] Module C:\Users\dbmilla\windbgnew\amd64\TTD\TTDRecordCPU.dll loaded
...lots of ".dll loaded" messages
[27:0] Module C:\Windows\System32\USER32.dll loaded
[28:0] Module C:\Windows\SYSTEM32\ntdll.dll loaded
[2d:0] Thread 2c5c created
[89:0] Thread 2ce0 created
[125:0] Thread 26ac created
...lots of ".dll loaded" messages
[174a:0] Thread 273c created 
[1a10:0] Thread 2e64 created
[1a39:0] Thread 2fe4 created 
[4caf:0] Thread 2378 created 
...lots of ".dll loaded" messages
This is the dbgeng.dll (WinDbg) REPL. To drop to Python3, press Ctrl-C. 
dbg>  
```

## Notes:
- `Step-over` has been expropriated to implement `step-back`

