
Breaking a File into Pieces with split :

The split [[command]] can split a ﬁle into two or more ﬁles. Unlike most of the text-manipulation commands described in this chapter, this command requires you to enter an output ﬁlename—or more precisely, an output ﬁlename preﬁx, to which is added an alphabetic code. You must also normally specify how large you want the individual ﬁles to be:

Split by Bytes The -b size or --bytes=size option breaks the input ﬁle into pieces of size bytes. This option can have the usually undesirable consequence of splitting the ﬁle
mid-line.

Split by Bytes in Line-Sized Chunks You can break a ﬁle into ﬁ les of no more than a speciﬁed size without breaking lines across ﬁles by using the -C=size or --line-bytes=size
option. (Lines will still be broken across ﬁles if the line length is greater than size .)

Split by Number of Lines The -l lines or --lines=lines option splits the ﬁle into chunks with no more than the speciﬁed number of lines.

As an example, consider breaking Listing 1.1 into two parts by number of lines:
```bash
$ split -l 2 listing1.1.txt numbers
```


The result is two ﬁles, numbersaa and numbersab , that together hold the original contents of listing1.1.txt .

If you don’t specify any defaults (as in split listing1.1.txt ), the result is output ﬁles split into 1,000-line chunks, with names beginning with x ( xaa , xab , and so on). If you don’t specify an input ﬁlename, split uses standard input.