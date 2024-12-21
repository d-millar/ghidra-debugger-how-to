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
