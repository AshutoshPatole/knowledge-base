[[command]]
Managing Process Priorities (NICE & RENICE):

Sometimes, you may want to prioritize your programs’ CPU use. For instance, you may be running a program that’s very CPU-intensive but that will take a long time to finish its work, and you don’t want that program to interfere with others that are of a more interactive nature. Alternatively, on a heavily loaded computer, you may have a job that’s more important than others that are running, so you may want to give it a priority boost. In either case, the usual method of accomplishing this goal is through the nice and renice commands. You can use nice to launch a program with a specified priority or use renice to alter the priority of a running program.

You can assign a priority to nice in any of three ways: 
- by specifying the priority preceded by a dash (this works well for positive priorities but makes them look like negative priorities), 
- by specifying the priority after a -n parameter
- by specifying the priority after an --adjustment= parameter. In all cases, these parameters are followed by the name of the program you want to run.

nice argument

For instance, the following three commands are all equivalent:
```sh
$ nice -12 number-crunch data.txt
$ nice -n 12 number-crunch data.txt
$ nice --adjustment=12 number-crunch data.txt
```
All three of these commands run the number-crunch program at priority 12 and pass it the data.txt fi le. If you omit the adjustment value, nice uses 10 as a default. The range of possible values is **–20 to 19**, with negative values having the highest priority. Only root may launch a program with increased priority (that is, give a negative priority value), but 
any user may use nice to launch a program with low priority. The default priority for a program run without nice is 0.

If you’ve found that a running process is consuming too much CPU time or is being swamped by other programs and so should be given more CPU time, you can use the renice program to alter its priority without disrupting the program’s operation. The syntax for renice is as follows:

```
$ renice priority [[-p] pids] [[-g] pgrps] [[-u] users]
```

You must specify the priority, which takes the same values this variable takes with nice. In addition, you must specify one or more PIDs (pids), one or more group IDs (pgrps), or one or more usernames (users). In the latter two cases, renice changes the priority of all programs that match the specified criterion—but only root may use renice in this way. Also, only root may increase a process’s priority. If you give a numeric value without a -p, -g, or -u option, renice assumes the value is a PID. You may mix and match these methods of specifi cation. For instance, you might enter the following command:

```
$ renice 7 16580 -u pdavison tbaker
```

This command sets the priority to 7 for PID 16580 and for all processes owned by pdavison and tbaker.