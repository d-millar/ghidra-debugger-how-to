# Adding a debugger

## Context
- OS: RHEL9
- debugger: drgn
- target: linux kernel/apps

## Debugger documentation

- Recommended reading: **drgn** (https://github.com/osandov/drgn)
- Also: **drgn (docs)** (https://drgn.readthedocs.io/en/latest/)

## Anatomy of a Ghidra debugger agent

- Currently, the Ghidra debugger has four "agents", i.e. servers capable of receiving information from a native debugger and passing it, frequently with modifications, to the Ghidra GUI.  They include the **dbgeng** agent that supports *Windows* debuggers, the **gdb** agent for *gdb* on a variery of platforms, the **lldb** agent for *macOS* and *Linux*, and the **jpda** agent for Java.  All but the last are written in Python 3, and all communicate with the GUI via a *protobuf*-based protocol described in *Ghidra/Debug/Debugger-rmi-trace*.  This blurb documents the addition of a fifth agent for *Meta*'s **drgn** debugger in the hopes of highlighting issues frequently encountered when creating an agent.

- At the highest level, each agent has four elements (ok, a somewhat arbitrary division, but...):
  - A set of launchers, often a mixture of *.bat/.sh* scripts and python
  - An XML schema
  - Python files for architecture, commands, hooks, methods, and common utility functions
  - Build logic

- Large portions of each are identical or similar across agents, so, as a general strategy, copying an existing agent and renaming all agent-specific variables, methods, etc. is not the worst plan of action, although typically this leads to large chunks of detritus that need to be edited out late in the development process.

## **Drgn** as an Example

### The first launcher

- The initial objective is to create a shell that sets ups the environment variables for parameters we'll need and invokes the target. For this project, I originally started duplicating the **lldb** agent and then switched to the **dbgeng** agent. Why? The hardest part of writing an agent is getting the initial launch pattern correct.  **Drgn** is itself written in python.  While gdb and lldb support python as scripting languages, their cores are not python-based. A python shell is invoked from within the *repl*. The dbgeng agent inverts this pattern, i.e. **pybag** is the main *repl* with the native *kd* interface exposed over COM as a scripting language. **Drgn** follows this pattern.

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

- For **drgn**, invoking the *drgn* command directly saves us a lot of the work involved in getting the environment correct.  We pass it our python launcher "local-drgn.py" instead of allowing it to call *run_interactive*, which does not return.  Instead, we created an instance of *prog* based on the parameters, complete the Ghidra-specific initialization, and call *run_interactive(prog)* ourselves.

- The python script needs to do the setup work for Ghidra and for **drgn**. A good start is to try to implement a script that calls the methods for connect, create, and start, with create doing as little as possible initially.  This should allow you to work the kinks out of "arch.py" and "util.py".

- For this particular target, there are some interesting wrinkles surrounding the use of *sudo* (required for most targets) which complicate where wheels are stored.  Additionally, the *-E* parameter is required to insure that the environment variable we defined get passed to the root environment. In the cases where we use *sudo*, the first message printed in the interactive shell with be the request for the user's password.

### The schema & the build logic

- The schema provides a basic structure for Ghidra's Model View and allows Ghidra to identify and locate various interfaces that are used to populate the GUI.  For example, the *Memory* interface identifies the container for items with the interface *MemoryRegion*, which provide information used to fill Memory View.  Among the important interfaces are *Process*, *Thread*, *Frame*, *Register*, *MemoryRegion*, *Module*, and *Section*.

- For the purposes of getting started, it's easiest to clone the **dbgeng** schema and modify it as needed.  Again, this will require substantial cleanup later on, but, as schema errors are frequently subtle and hard to identify, revisiting is probably the better approach.  Similarly, "build.gradle" can essentially be cloned from **dbgeng**, with the appropriate change to *eclipse.project.name*. The *.tlb* logic is not relevant and can be trimmed out immediately, but we'll postpone other changes until later.

### The Python files

- At this point, we can start actually implementing the **drgn** agent. **Arch.py** is usually a good starting point, as much of the initial logic depends on it. For "arch.py", the hard bit is knowing what maps to what. The *language_map* converts the debugger's self-reported architecture to Ghidra's language set. Ghidra's languages are mapped to a set of language-to-compiler maps, which are then used to map the debugger's self-reported language to Ghidra's compiler. Certain combinations are not allowed because Ghidra has no concept of that language-compiler combination.  For example, x86 languages never map to "default".  Hence, the need for a *x86_compiler_map*, which defaults to something else (in this case, *gcc*).

- After "arch.py", a first pass at **util.py** is probably warranted. In particular, the version info is used early in the startup process.  A lot of this code is not relevant to our current project, but at a minimum we want to implement (or fake out) methods such as *selected_process*, *selected_thread*, and *selected_frame*. In this example, there probably won't be more than one session or one process. Ultimately, we'll have to decide whether we even want *Session* in the schema. For now, we're defaulting session and process to 0, and thread to 1, as 0 is invalid for debugging the kernel. (Later, it becomes obvious that the attached pid and *prog.main_thread().tid* make sense for user-mode debugging, and *prog.crashed_thread().tid* makes sense for crash dump debugging.)

- With "arch.py" and "util.py" good to a first approximation, we would normally start implementing "put" methods in **commands.py** for various objects in the Model View, starting at the root of the tree and descending through the children. Again, *Session* and *Process* are rather poorly-defined, so we skip them (leaving one each) and tackle *Threads*. Typically, for each iterator in the debugger API, two commands get implemented - one that does the actual work, e.g. *put_threads()* and one that wraps this method in a (potentialy batched) transaction, e.g. *ghidra_trace_put_threads()*. The working method typically creates the path to the container using patterns for the container, individual keys, and the combination, e.g. *THREADS_PATTERN*, *THREAD_KEY_PATTERN*, and *THREAD_PATTERN*.  Patterns are built up from other patterns, going back to the root.  A trace object corresponding to the debugger object is created from the path and inserted into the trace.

- Once this code has been tested, attributes of the object can be added to the base object using *set_value*. Attributes that are not primitives can be added either immediately using *create_object* with extensions to the path, populated, and *insert*ed, or placeholders can be created using just the *create_object* method. The advantage to using a placeholder is that expensive methods can be postponed until the user requests that information, with the downside that *method*s must be added to populate those nodes.

- Having completed a single command, we can proceed in one of two directions - we can continue implementing commands for other objects in the tree or we can implement matching "refresh* methods in **methods.py** for the completed object. "Methods.py" also requires patterns which are used to match a path to a trace object, usually via *find_x_by_pattern* methods. The *refresh* methods may or may not rely on the *find_by* methods depending on whether the matching command needs parameters.  For example, we may want to assume the *selected_thread* matches the current object in the view, in which case it can be used to locate that node, or we may want to force the method to match on the node if the trace object can be easily matched to the debugger object, or we may want to use the node to set *selected_thread*.

- *Refresh* methods (and others) are often annotated in several ways.  The *@REGISTRY.method* annotation makes the method available to the GUI.  It specifies the *action* to be taken and the *display* that appears in the GUI pop-up menu. *Actions* may be purely descriptive or may correspond to built-in actions taken by the GUI, e.g. 'refresh' and many of the control methods, such as 'step_into'. Parameters for the methods maybe annotated with *sch.Schema* (on the first parameter) to indicate the nodes to which the method applies, and with *ParamDesc* to describe the parameter's type and label for pop-up dialogs. After retrieving necessary parameters, *refresh* methods invoke methods from "commands.py" wrapped in a transaction.

- For *drgn*, we implemented *put*/*refresh* methods for threads, frames, registers (*putreg*), and local variables, then modules and sections, memory and regions, the environment, and finally processes.  We also implemented *putmem* using the debugger's *read* API. *Symbols* was another possibility, but, for the moment, populating symbols seemed to expensive. Instead, *retrieve_symbols* was added to allow per-pattern symbols to be added. Unfortunately, the *drgn* API doesn't support wildcards, so eventually some other strategy will be necessary.

- The remaining set of python functions, **hooks.py**, comprises callbacks for various events sent by the native debugger. The current *drgn* code has no event system. A set of skeletal methods has been left in place as (a) we can use the single-step button as a stand-in for "update state", and (b) some discussion exists in the *drgn* user forums regarding eventually implementing more control functionality.

### Revisiting the schema

- At this point, revisiting and editing the schema may be called for.  For example, for **drgn**, it's not obvious that there can ever be more than one session, so it may be cleaner to embed *Processes* at the root. This, in turn, requires editing the "commands.py" and "methods.py" patterns. Similarly, as breakpoints are supported, the breakpoint-related entries may safely be deleted.

- In general, the schema can be structured however you like, but there are several details worth mentioning. Interfaces generally need to be respected for various functions in the GUI to work. Process, thread, frame, module, section, and memory elements can be named arbitraily, but their interfaces must be named correctly.  Additionally, the logic for finding elements in the tree is quite complicated. Elements to be traversed as part of the default search process must be tagged *canonical* and containers should have the interface *Aggregate*. 

- Each entry may have *elements* of the same type are ordered by keys, and *attributes* of arbitrary type. The *element* entry describes the schema for all elements; the schema for attributes may be given explicit using name *attribute* entries or defaulted using the unnamed *attribute* entry, typically *<attribute schema="VOID">* or *<attribute schema="ANY">*. The schema for any element in the Model View is visible using the hover, which helps substantially when trying to identify schema traversal errors.

- Schema entries may be marked *hidden=yes* with the obvious result. Additionally, certain attribute names and schema have special properties.  For example, *_display* defined the visible ID for an entry in the Model tree, and *ADDRESS* and *RANGE* mark attributes which are navigable.


### Unit tests 

### Documentation

### Extended features

 






