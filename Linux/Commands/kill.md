
[[command]]

Sometimes, reducing a process’s priority isn’t a strong enough action. A program may have become totally unresponsive, or you may want to terminate a process that shouldn’t be running. In these cases, the kill command is the tool to use. This program sends a signal (a method that linux uses to communicate with processes) to a process. The signal is usually sent by the kernel, the user, or the program itself to terminate the process. Linux supports many numbered signals, each of which is associated with a specific name. You can see them all by typing kill -l. If you don’t use -l, the syntax for kill is as follows:

```bash
kill -s pid
```

The -s signal parameter sends the specified signal to the process. You can specify the signal using either a number (such as 9) or a name (such as SIGKILL). The signals you’re most likely to use are
- 1 (SIGHUP, which terminates interactive programs and causes many daemons to reread their configuration files), 
- 9 (SIGKILL, which causes the process to exit without performing routine shutdown tasks) and,
- 15 (SIGTERM, which causes the process to exit but allows it to close open files and so on). If you don’t specify a signal, the default is `15 (SIGTERM)`.

 You can also use the shortened form -signal. If you do this and use a signal name, you should omit the SIG portion of the name—for instance, use KILL rather than SIGKILL. The pid option is, of course, the PID for the process you want to kill. You can obtain this number from ps or top.

A variant on kill is killall, which has the following form:

```bash
killall [options] [--] name [...]
```

This command kills a process based on its name rather than its PID number. For instance, killall vi kills all the running processes called vi. You may specify a signal in the shortened form (-signal) or by preceding the signal number with -s or --signal. One potentially important option to killall is -i, which causes it to ask for confirmation before sending the signal to each process. You might use it like this:

```bash
$ killall -i vi

Kill vi(13211) ? (y/n) y
Kill vi(13217) ? (y/n) n
```

In this example, two instances of the Vi editor were running, but only one should have been killed. As a general rule, if you run killall as root, you should use the -i parameter; 
if you don’t, it’s all too likely that you’ll kill processes that you shouldn’t, particularly if the computer is being used by many people at once.

You should never use -9 when killing a process.
It doesn't give the process a chance to cleanly:

1. Shut down socket connections
2. Clean up temp files
3. Inform its children that it is going away
4. Reset its terminal characteristics

>[!attention]
Use `Kill(15)` while killing a process.
Don't use **`kill -9`**. 

Don't bring out the combine harvester just to tidy up the flower pot.

#shell 