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
- implement version info in util.py
