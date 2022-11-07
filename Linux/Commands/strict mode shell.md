[[command]]
Often times when we write scripts it is hard to debug bugs because scripts do not work as we think they should.
For example, take any programming language(I'm assuming python here) if any error occurs inside the program it halts immediately without running next lines like

```python
print("Hello")
print("world)     # error here
print("!")
```
We got this when compiled 
```
File "<string>", line 4
    print("world)
                ^
SyntaxError: EOL while scanning string literal
```
## Problem 1
Now let's look what bash script does here
```bash title="hello.sh"

#!/bin/bash
echo "hello"
failHere=$(somethingToFail)  # this should fail
echo "world!"
```
As you can see this should definitely fail in second line and `echo "world!"` should not be printed but it does.
```bash
[thor@marvel-studios ~]$ ./hello.sh 
hello
./hello.sh: line 4: somethingToFail: command not found
world!
```
This shouldn't be the perfect way of writing scripts. What if there was a command which depended on previous execution state? It will get executed whereas it shouldn't. And most importantly even if the script had an error it returns a `0` result code which means everything went fine.

```bash
[thor@marvel-studios ~]$ ./hello.sh 
hello
./hello.sh: line 4: somethingToFail: command not found
world!
[thor@marvel-studios ~]$ echo $?
0
```
There are multiple things which should be taken care of while writing scripts. I have added some of the scenarios with solutions to avoid them.

### 1. set - The saviour

We looked the first problem above and in order to solve it we have to use `set` lines. These lines deliberately cause your script to fail. Believe me, this is a good thing. With these settings, certain common errors will cause the script to immediately fail, explicitly and loudly. Otherwise, you can get hidden bugs that are discovered only when they blow up in production.

The `set -e` option instructs bash to immediately exit if any command [1] has a non-zero exit status. You wouldn't want to set this for your command-line shell, but in a script it's massively helpful. 

```bash title="hello.sh"

#!/bin/bash
set -e
echo "hello"
failHere=$(somethingToFail)  # this should fail
echo "world!"
```
Now this solves the 2 mentioned problems 
1. It does not run lines after it encounter errors.
2. It does not returns a 0 exit code

```bash
[thor@marvel-studios ~]$ ./hello.sh 
hello
./hello.sh: line 5: somethingToFail: command not found
[thor@marvel-studios ~]$ echo $?
127
```
(127 exit code means command not found)

## Problem 2
Sometimes we are in a too much hurry that we either forget to declare a variable or we mis-spell it which is kinda hard to identify if the variable is used in the form of `string interpolation`. This is where we get our second problem. For example 

```bash title="variable.sh"


#!/bin/bash
firstName="Thor"
fullName="$firstname, the Odin's son"
echo "$fullName"
```
Just take a moment here and see where did the script went wrong and what do you think will happen if we run this script?


```bash
[thor@marvel-studios ~]$ ./variable.sh 
, the Odin's son
```
Now you would have got the bug. The script ran fine but we misspelled the `firstName` camelCase variable as `firstname` all lowercase variable.

### set -u

`set -u` affects variables. When set, a reference to any variable you haven't previously defined is an error, and causes the program to immediately exit. Languages like Python, C, Java and more all behave the same way, for all sorts of good reasons. 
Now let's change the script a bit and look how it performs.

```bash title="variable.sh"

#!/bin/bash
set -e
set -u
firstName="Thor"
fullName="$firstname, the Odin's son"
echo "$fullName"
```
```
[thor@marvel-studios ~]$ ./variable.sh 
./variable.sh: line 5: firstname: unbound variable
```
Yeah kind of unhelpful message for some reasons but it is better than a silent bug.

:::info
**set -u** does not exits out if we have `$*` and `$@` in our scripts - they are exceptions in  -u.

But if you use named arguments as `$1` and `$2` in script then they will **throw an error**
:::

## Problem 3
Remember in problem 1 we saw exit code returned as 0 even if there was an error and we fixed it using set -e.
Well set -e option **does not** provide protection if there is a **pipe** character involved in the command.

```bash title="pipe.sh"

#!/bin/bash

grep some-string /non/existent/file | sort
```
Output: 
```bash
[thor@marvel-studios ~]$ ./pipe.sh 
grep: /non/existent/file: No such file or directory
[thor@marvel-studios ~]$ echo $?
0
```

This indeed failed but returns a 0 exit code. Here, grep has an exit code of 2, writes an error message to `stderr`, and an empty string to `stdout`. This empty string is then passed through `sort`, which happily accepts it as valid input, and returns a status code of 0. This is fine for a command line, but bad for a shell script: you almost certainly want the script to exit right then with a nonzero exit code. 


### set -o pipefail

This setting prevents errors in a pipeline from being masked. If any command in a pipeline fails, that return code will be used as the return code of the whole pipeline. By default, the pipeline's return code is that of the last command - even if it succeeds.

```bash title="pipe.sh"

#!/bin/bash
set -o pipefail
grep some-string /non/existent/file | sort
```
This will stop the execution when it finds that file does not exists and never passes the ouutput to the sort command
```bash
[thor@marvel-studios ~]$ ./pipe.sh 
grep: /non/existent/file: No such file or directory
[thor@marvel-studios ~]$ echo $?
2
```

## Disable strict mode temporarily
Sometimes your script needs to source a file that doesn't work with your strict mode. What then?

```bash
source /path-to-file/file.env
# Your strict-mode script immediately exits here,
# with a fatal error.
```
The solution is to
1. Temporarily disable that aspect of strict mode.
2. Source the document.
3. Re-enable, on the next line.

The most common time you'll need this will be when the document references an undefined variable. Temporarily allow such a transgression with **set +u**:

```bash
set +u
source some/bad/file.env
set -u
```
(Remember, set +u **disables** this variable strictness, and set -u **enables** it. A bit counterintuitive)
Same applies for set -e and set -o. There are few more options which I don't think are that neccessary, maybe sometime in the future I will update this document if I encounter any other use case.

:::tip
For scripts having many number of lines it is often better option to select **python** for scripting because bash is slow and error prone and since python actually ships preinstalled with most of the distributions it makes sense to use it.
:::


#shell 