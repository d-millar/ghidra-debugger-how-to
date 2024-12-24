# Adding a debugger

## Context
- OS: RHEL9
- debugger: drgn
- target: linux kernel/apps

## Debugger documentation

- Recommended reading: **drgn** (https://github.com/osandov/drgn)
- Also: **drgn (docs)** (https://drgn.readthedocs.io/en/latest/)

## My Notes (as implemented)

- duplicate one of the dirs in Ghidra/Debug; rename everything.
- implement arch.py
  - The hard bit is knowing what maps to what. The language_map converts the debugger's self-reported architecture to Ghidra's language set. Ghidra's languages are mapped to a set of language-to-compiler maps, which are then used to map the debugger's self-reported language to Ghidra's compiler. Certain combinations are not allowed because Ghidra has no concept of that language-compiler combination.  For example, x86 languages never map to "default".  Hence, the need for a x86_compiler_map, which defaults to something else (in this case, "gcc").
- implement version info in util.py
- implment a debugger-launcher
  - In this specific case, the underlying debugger is implemented in python, so the dbgeng pattern is probably a good match, although we're focused here on Linux as the host OS.  Basically, we're going to create a shell that sets ups the environment variables that we're going to use and invokes python with a target python script.
  - The python script needs to do the setup work for Ghidra and for drgn. A good start is to try to implement a script that calls the methods for connect, create, and start, with create doing as little as possible initially.  This should allow you to work the kinks out of arch.py and util.py.
  - For this particular target, there are some interesting wrinkles surrounding the use of "sudo" (required for most targets) which complicate where wheels are stored, and surrounding the implementation of drgn.cli.\_main().  "\_main" does not return we're invoking "drgn" with a script, namely local_drgn.py, which allows us to instantiate "prog", then set up ghidra, then invoke the drgn repl.
- implement something simple near the top of the objects tree
  - In this case, am starting with threads, given the meaning of processes are not obvious to me at the moment. This will typically highlight errors you've made in your launcher, e.g. I forgot to add the logic to include symbols causing the thread iterator to fail.
