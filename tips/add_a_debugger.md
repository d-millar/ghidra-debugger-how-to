# Adding a debugger

## Context
- OS: RHEL9
- debugger: drgn
- target: linux kernel/apps

## Debugger documentation

- Recommended reading: **drgn** (https://github.com/osandov/drgn)
- Also: **drgn (docs)** (https://drgn.readthedocs.io/en/latest/)

## Anatomy of a Ghidra debugger agent

- Currently, the Ghidra debugger has four "agents", i.e. servers capable of receiving information from a native debugger and passing it, frequently with modifications, to the Ghidra GUI.  They include the **dbgeng** agent that supports *Windows* debuggers, the **gdb** agent for *gdb* on a variery of platforms, the **lldb** agent for *macOS* and *Linux*, and the **jpda** agent for Java.  All but the last are written in Python 3, and all communicate with the GUI via a protobuf-based protocol described in *Ghidra/Debug/Debugger-rmi-trace*.  This blurb documents the addition of a fifth agent for *Meta*'s **drgn** debugger in the hopes of highlighting issues frequently encountered when creating an agent.

- At the highest level, each agent has four elements (ok, a somewhat arbitrary division, but...):
1. A set of launchers, often a mixture of *.bat/.sh* scripts and python
2. An XML schema
3. Python files for architecture, commands, hooks, methods, and common utility functions
4. Build logic

- Large portions of each are identical or similar across agents, so, as a general strategy, copying an existing agent and renaming all agent-specific variables, methods, etc. is not the worst plan of action, although typically this leads to large chunks of detritus that need to be edited out late in the development process.

## **Drgn** as an Example

### The first launcher

- The initial objective is to create a shell that sets ups the environment variables for parameters we'll need and invokes the target. For this project, we originally started duplicating the **lldb** agent and then switched to the **dbgeng** agent. Why? The hardest part of writing an agent is getting the initial launch pattern correct.  **Drgn** is itself written in python.  While gdb and lldb support python as scripting languages, their cores are not python-based. A python shell is invoked from within the *repl*. The dbgeng agent inverts this pattern, i.e. we use **pybag** as the main *repl* and the native *kd* interface exposed over COM as a scripting language. **Drgn** follows this pattern.

- That said, a quick look at the launchers in the **dbgeng** project (under "data/debugger-launchers") shows *.bat* files, each of which calls a python file in "data/support". As **drgn** is a Linux-only debugger, we need to convert the *.bat* examples to *.sh*. Luckily, the conversion is pretty simple: most line annotations use *#* in place of *::* and environment variable are referenced using *$VAR* in place of *%VAR%*.

- The syntax of the *.sh* is typical of any *\*nix* shell. Our **drgn** launchers include:
  - A *#!* line for the shell invocation
  - The Ghidra license
  - A *#@title* line for the launcher name
  - An *#@desc*-annotated HTML description
  - *#@menu-group* for separating launchers by type
  - *#@icon* for an icon
  - *#@help* pointing to a help file
  - Some number of *#@env* variables referenced by the python code
-The *#@env* lines are composed of:
  - The variable name (usually in caps)
  - colon-separated, the variable type
  - *!=* or *=* for a default value (required or not)
  - The label for a dialog if the user needs to be queried
  - A description

- For **drgn**, invoking the *drgn* command directly saves us a lot of the work involved in getting the environment correct.  We pass it our python launcher "local-drgn.py" instead of allowing it to call *run\_interactive*, which does not return.  Instead, we create an instance of *prog* based on the parameters, complete the Ghidra-specific initialization, and call *run\_interactive(prog)* ourselves.

- The python script needs to do the setup work for Ghidra and for **drgn**. A good start is to try to implement a script that calls the methods for connect, create, and start, with create doing as little as possible initially.  This should allow you to work the kinks out of "arch.py" and "util.py".

- For this particular target, there are some interesting wrinkles surrounding the use of *sudo* (required for most targets) which complicate where wheels are stored.  Additionally, the *-E* parameter is required to insure that the environment variable we defined get passed to the root environment. In the cases where we use *sudo*, the first message printed in the interactive shell with be the request for the user's password.

### The schema & the build logic

- The schema provides a basic structure for Ghidra's Model View and allows Ghidra to identify and locate various interfaces that are used to populate the GUI.  For example, the *Memory* interface identifies the container for items with the interface *MemoryRegion*, which provide information used to fill Memory View.  Among the important interfaces are *Process*, *Thread*, *Frame*, *Register*, *MemoryRegion*, *Module*, and *Section*.

- For the purposes of getting started, it's easiest to clone the **dbgeng** schema and modify it as needed.  Again, this will require substantial cleanup later on, but, as schema errors are frequently subtle and hard to identify, revisiting is probably the better approach.  Similarly, "build.gradle" can essentially be cloned from **dbgeng**, with the appropriate change to *eclipse.project.name*. The *.tlb* logic is not relevant ant can be trimmed out immediately, but we'll postpone other changes until later.

### The Python files

- At this point, we can start actually implementing the **drgn** agent. "Arch.py" is usually a good starting point, as much of the initial logic depends on it. For "arch.py", the hard bit is knowing what maps to what. The *language\_map* converts the debugger's self-reported architecture to Ghidra's language set. Ghidra's languages are mapped to a set of language-to-compiler maps, which are then used to map the debugger's self-reported language to Ghidra's compiler. Certain combinations are not allowed because Ghidra has no concept of that language-compiler combination.  For example, x86 languages never map to "default".  Hence, the need for a *x86\_compiler\_map*, which defaults to something else (in this case, *gcc*).

- After "arch.py", a first pass at "util.py" is probably warranted. In particular, the version info is used early in the startup process.




## My Notes (as implemented)

- duplicate one of the dirs in Ghidra/Debug; rename everything.
- implement arch.py
  - 
- implement version info in util.py
- implement a debugger-launcher
  - In this specific case, the underlying debugger is implemented in python, so the dbgeng pattern is probably a good match, although we're focused here on Linux as the host OS (i.e. we'll need to switch the env logic from that for a .bat to a .sh).  
- implement something simple near the top of the objects tree
  - In this case, am starting with threads, given the meaning of processes are not obvious to me at the moment. This will typically highlight errors you've made in your launcher, e.g. I forgot to add the logic to include symbols causing the thread iterator to fail.
- implement refresh_threads
  - Assuming put_threads works correctly, you may want to implement the matching refresh method in methods.py.  Be cognizant that, if the method is triggered from the objects tree, any error messages may not appear in the  terminal. Hovering on the object will indicate the object type which should match the annotations on the method.  If not, check the schema from the root down.
- implement registers
  - Keep in mind trace.put_registers expects bytes
- implement memory
  -- something about mapping strategy
- something about methods with parameters, required parameters & dialog
- something about implicit functions like refresh
- explain the .bat/.sh syntax

-implement other launchers
-implement putmem_state
