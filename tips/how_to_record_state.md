# recording state at given breakpoint for later processing

- for each target, set the breakpoint first
  
- lldb:
  - `br com add 1`  (assuming bpt#==1)
  - `script ghidralldb.hooks.on_stop(None)`
  - `cont`
  - `DONE` (to exit)
 
- gdb:
  - `commands` (takes the last by default)
  - `script ghidragdb.hooks.on_stop(None)`
  - `cont`
  - `end` (to exit)
 
- dbgeng:
  - Find the breakpoint in the Model View
  - Ctrl-right -> `Add Handler`
  - Handler: `on_stop`
  - Use the terminal to set the command to "g"
    or
  - Set the original breakpoint a la `bp 0xdeadbeef "g"`
