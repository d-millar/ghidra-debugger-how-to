# macos kernel

## Context
- OS: macos x64 with VMware
- debugger: lldb
- target: Sequoia

## Set-up

- Recommended reading: **Debugging macOS Kernel using Virtual Box** (https://klue.github.io/blog/2017/04/macos_kernel_debugging_vbox/)
- Also: **How to Convert a MacOS Installer to ISO** (https://osxdaily.com/2020/07/20/how-convert-macos-installer-iso)
- Also: **Install macOS does not appear to be a valid OS installer application** (https://apple.stackexchange.com/questions/439402/install-macos-monterey-app-does-not-appear-to-be-a-valid-os-installer-applicatio)

- From terminal:
```
softwareupdate --list-full-installers
softwareupdate --fetch-full-installer --full-installer-version 15.1
hdiutil create -o /tmp/Sequoia -size 20000m -volname Sequoia -layout SPUD -fs HFS+J
hdiutil attach /tmp/Sequoia.dmg -noverify -mountpoint /Volumes/Sequoia
sudo /Applications/Install\ macOS\ Sequoia.app/Contents/Resources/createinstallmedia --volume /Volumes/Sequoia --nointeraction
hdiutil detach /Volumes/Install\ macOS\ Sequoia
hdiutil convert /tmp/Sequoia.dmg -format UDTO -o ~/Desktop/Sequoia.cdr
 mv ~/Desktop/Sequoia.cdr ~/Desktop/Sequoia.iso
```
- In VMware Fusion, create new VM
- Boot the VM and issue the command:
```
sudo nvram boot-args="-v debug=0x144"
```
- From the Ghidra toolbar, `Configure and launch <anything> using...->dbgeng-kernel`.
```
Python command: py (typically)
Argument: exdi:CLSID={29f9906e-9dbe-4d4b-b0fb-6acf7fb6d014},Kd=Guess,DataBreaks=Exdi
Type: EXDI
Use dbgmodel: either, but dbgmodel is better
Path to dbgeng.dll directory: C:\Program Files (x86)\Windows Kits\10\Debuggers\x64 (typically)
```
- `Launch`
- at the`Launch Failed` timeout dialog, choose `Keep`

## Terminal Output:

```
```

## Notes:

