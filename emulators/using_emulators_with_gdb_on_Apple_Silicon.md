# How to debug an emulator with a gdbstub on aarch64 macos

Option one is, of course, use lldb.  This may or may not work, and/or may or may not be in your comfort zone.  Given the current difficulties involved building gdb, esp. gdb-multiarch, on Apple Silicon, using gdb-multiarch directly may not be an option.  If, however, you have a running Linux VM, for example in VMware or QEMU, you have an alternative.  The basic idea is to modify the gdb-over-ssh shell to circle back to an emulator running natively on macos.

To do this, you'll need to:
- ssh into your running Linux VM (both to insure things are installed and make sure ssh is set up correctly)
- `apt install gdb-multiarch'
- verity the version of python used inside of gdb-multiarch (e.g. `pi`->`import sys`->`sys.version`)
- assuming that version is the default in your shell, `python3 -m pip install ghidratrace` & `python3 -m pip install ghidragdb`
- if needed, `python3 -m pip install psutil`

At this point, you'll probably want to modify an existing debugger-launcher for future use.  We give an example using ssh-gdb.sh:

```
#!/usr/bin/bash
## ###
#  IP: GHIDRA
...
##
#@timeout 60000
#@title gdb via ssh
#@desc <html><body width="300px">
#@desc   <h3>Launch with <tt>gdb</tt> via <tt>ssh</tt></h3>
#@desc   <p>
#@desc     This will launch the target on a remote machine using <tt>gdb</tt> via <tt>ssh</tt>.
#@desc     For setup instructions, press <b>F1</b>.
#@desc   </p>
#@desc </body></html>
#@menu-group remote
#@icon icon.debugger
#@help TraceRmiLauncherServicePlugin#gdb_ssh
#@enum StartCmd:str run start starti
#@arg :str "Image" "The target binary executable image on the remote system"
#@args "Arguments" "Command-line arguments to pass to the target"
#@env OPT_HOST:str="localhost" "[User@]Host" "The hostname or user@host"
#@env OPT_REMOTE_PORT:int=12345 "Remote Trace RMI Port" "A free port on the remote end to receive and forward the Trace RMI connection."
#@env OPT_EXTRA_SSH_ARGS:str="" "Extra ssh arguments" "Extra arguments to pass to ssh. Use with care."
#@env OPT_GDB_PATH:str="gdb" "gdb command" "The path to gdb on the remote system. Omit the full path to resolve using the system PATH."
#@env OPT_START_CMD:StartCmd="starti" "Run command" "The gdb command to actually run the target."

target_image="$1"
shift
target_args="$@"

ssh "-R$OPT_REMOTE_PORT:$GHIDRA_TRACE_RMI_ADDR" -t $OPT_EXTRA_SSH_ARGS "$OPT_HOST" "TERM='$TERM' '$OPT_GDB_PATH' \
  -q \
  -ex 'set pagination off' \
  -ex 'set confirm off' \
  -ex 'show version' \
  -ex 'python import ghidragdb' \
  -ex 'file \"$target_image\"' \
  -ex 'set args $target_args' \
  -ex 'ghidra trace connect \"localhost:$OPT_REMOTE_PORT\"' \
  -ex 'ghidra trace start' \
  -ex 'ghidra trace sync-enable' \
  -ex '$OPT_START_CMD' \
  -ex 'set confirm on' \
  -ex 'set pagination on'"
```

You'll want to modify the shell, the command line args, and a few of the gdb commands.  Specifically, you'll probably want:
- `#!/usr/bin/bash` -> `#!/bin/bash`
- `#@title gdb via ssh` -> `#@title gdb via ssh and back` or something
- add an argument reflecting the emulator's gdb port to the Linux VM, e.g.
```
ssh "-R$OPT_REMOTE_PORT:$GHIDRA_TRACE_RMI_ADDR" -t $OPT_EXTRA_SSH_ARGS "$OPT_HOST" "TERM='$TERM' '$OPT_GDB_PATH' \
``
could be changed to 
```
ssh "-R$OPT_REMOTE_PORT:$GHIDRA_TRACE_RMI_ADDR" -R 2345:localhost:2345 -t $OPT_EXTRA_SSH_ARGS "$OPT_HOST" "TERM='$TERM' '$OPT_GDB_PATH' \
```
- delete the gdb commands:
```
  -ex 'file \"$target_image\"' \
  -ex 'set args $target_args' \
  -ex '$OPT_START_CMD' \
```
- add the commands:
`  -ex 'set arch armv4t' \` say, after `set confirm off`
and
`  -ex 'target remote :2345' \` after `  -ex 'python import ghidragdb' \`

Depending on the emulator, you'll probably need to do one or more of the following after launching:
- From Modules, `Modules->Map the current trace to the current program using identical addresses (<=>)` or equivalent.
- From Regions, `Force full`.
- From the terminal, step once (`stepi`) to insure a sane state.

