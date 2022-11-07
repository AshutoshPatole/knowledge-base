
Cat [[command]] :

The cat command’s name is short for concatenate, and this tool does just that: It links together an arbitrary number of ﬁles end to end and sends the result to standard output. By combining cat with output redirection, you can quickly combine two ﬁles into one: 

```
$ cat first.txt second.txt > combined.txt
```

Although cat is ofﬁcially a tool for combining ﬁles, it’s also commonly used to display the contents of a short ﬁle. If you type only one ﬁlename as an option, cat displays that ﬁle. This is a great way to review short ﬁles; but for long ﬁles, you’re better off using a full-ﬂedged pager command, such as more or less .

You can add options to have cat perform minor modiﬁcations to the ﬁles as it combines them:

Display Line Ends If you want to see where lines end, add the -E or --show-ends option. The result is a dollar sign ( $ ) at the end of each line.

Number Lines The -n or --number option adds line numbers to the beginning of every line. The -b or --number-nonblank option is similar, but it numbers only lines that contain text.

Minimize Blank Lines The -s or --squeeze-blank option compresses groups of blank lines down to a single blank line.

Display Special Characters The -T or --show-tabs option displays tab characters as ^I .

The -v or --show-nonprinting option displays most control and other special characters using carat ( ^ ) and M- notations.

The tac command is similar to cat , but it reverses the order of lines in the output.