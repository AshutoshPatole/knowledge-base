

## Streams, Redirection, and Pipes

To begin understanding redirection and pipes, you must ﬁrst understand the different types of input and output streams. Three are most important for this topic:

- Standard Input Programs accept keyboard input via standard input, or stdin. In most cases, this is the data that comes into the computer from a keyboard.

- Standard Output Text-mode programs send most data to their users via standard output (aka stdout), which is normally displayed on the screen, either in a full-screen text-mode
session or in a GUI window such as an xterm .

- Standard Error Linux [[command]] provides a second type of output stream, known as standard error, or stderr. This output stream is intended to carry high-priority information such as error
messages.

## Redirecting Input and Output

```bash
[thor@marvel-studios ~]$ echo $NNTPSERVER > nntpserver.txt
```

The result is that the file nntpserver.txt contains the output of the command (in this case, the value of the `$NNTPSERVER` environment variable)



## Redirection operator effect

|Operator|Description|
|---|---|
|>|  Creates a new file containing standard output. If the specified file exists, it’s overwritten.|
|>>|  Appends standard output to the existing file. If the specified file doesn’t exist, it’s created.|
|2>| Creates a new file containing standard error. If the specified file exists, it’s overwritten.|
|2>>|  Appends standard error to the existing file. If the specified file doesn’t exist, it’s created.|
|<| Sends the contents of the specified file to be used as standard input.|
|<<| Accepts text on the following lines as standard input.|
|&>| Creates a new file containing both standard output and standard error. If the specified file exists, it’s |overwritten.|

NOTE : A common trick is to redirect standard output or standard error to /dev/ null . This file is a device that’s connected to nothing; it’s used when you want to get rid of data. For instance, if the whine program is generating too many error messages, you can type `whine 2> /dev/null` to run it and discard its error messages.

## Piping Data Between Programs

A pipe redirects the first program’s standard output to the second program’s standard input and is denoted by a vertical bar ( | ):

```bash
$ first | second
```

For instance, suppose that first generates some system statistics, such as system uptime, CPU use, number of users logged in, and so on. This output might be lengthy, so you want
to trim it a bit. You might therefore use second , which could be a script or command that echoes from its standard input only the information in which you’re interested. 

Pipes can be used in sequences of arbitrary length:

```bash
$ first | second | third | fourth | fifth | sixth [...]
```

## Generating Command Lines XARGS

Sometimes you’ll ﬁnd yourself constructing a series of commands that are similar to each other but not similar enough to enable you to use their normal options to substitute a single
command. For instance, suppose you want to remove every file in a directory tree with a name that ends in a tilde ( ~ ). (This filename convention denotes backup files created by
certain text editors.) With a large directory tree, this task can be daunting;

The xargs command builds a command from its standard input. The basic syntax for this command is as follows:

```bash
xargs [options] [command [initial-arguments]]
```

The command is the command you want to execute, and initial-arguments is a list of arguments you want to pass to the command. The options are xargs options; they aren’t
passed to command . When you run xargs , it runs command once for every word passed to it on standard input, adding that word to the argument list for command . If you want to pass
multiple options to the command, you can protect them by enclosing the group in  quotation marks.

For instance, consider the task of deleting all those backup files, denoted by tilde characters. You can do this by piping the output of find to xargs , which then calls rm :

```bash
[thor@marvel-studios ~]$ find ./ -name “*~” | xargs -d “\n” rm
```

The first part of this command `( find ./ -name “*~” )` finds all the files in the current directory ( ./ ) or its subdirectories with a name that ends in a tilde `( *~ )`. This list is then piped to xargs , which adds each input value to its own rm command. Problems can arise if ﬁlenames contain spaces, since by default xargs uses both spaces and newlines as item delimiters. The -d “\n” option tells xargs to use only newlines as delimiters, thus avoiding this problem in this context. (The find command separates each found filename with a newline.)


#shell 