# lldb tips

- launchers probably require `!/bin/bash` vs. `!/usr/bin/env bash`.
- equivalents:
  - `target remote :12345` => `gdb-remote localhost:12345`
  - `set debug remote 1` => `log enable gdb-remote packets`
  - `show arch` => `settings show target.default-arch`
