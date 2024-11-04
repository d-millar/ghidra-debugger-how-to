# Android (java)

## Context
- OS: macos arm64
- debugger: Ghidra JDI
- target: AndroidStudio (emulator) Java targets

## Set-up

- Generating a test file: 
	- `javac HelloWorld.java`
	- `~/Library/Android/sdk/build-tools/35.0.0/d8  --output HelloWorld.jar HelloWorld.class`
	- `adb push ./HelloWorld.jar /data/local/tmp`
	- test using `adb shell dalvikvm -cp /data/local/tmp/HelloWorld.jar HelloWorld`
	- start and get pid (am assuming the program doesn't terminate)
	- `adb forward tcp:54321 jdwp:pid`
- From the Ghidra toolbar, `Configure and launch target using... -> java attach port` 
```
Arch: Dalvik
Host: localhost
Port: 54321
Timeout: 0
JShell cmd (if desired): /usr/bin/jshell
```

## Terminal Output:

```
INFO  Using log config file: file:/Path_to/generic.log4jdev.xml   (LoggingInitialization.java:50) 
INFO  Using log file: /Path_to/application.log   (LoggingInitialization.java:51) 
INFO  Loading user preferences: /Path_to/preferences   (Preferences.java:122) 
INFO  Searching for classes...   (ClassSearcher.java:409) 
INFO  Class search complete (1341 ms)   (ClassSearcher.java:135) 
|  Welcome to JShell -- Version 21.0.3
|  For an introduction type: /help intro

jshell> Connected to jdi at /127.0.0.1:58257
```

## Errors:
- For most processes, you need to hit `Interrupt` to get a sane state.
