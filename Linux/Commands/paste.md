
Merging Lines with paste :
similar to [[join]]
The paste command merges ﬁles line by line, separating the lines from each ﬁle with tabs.

```
$ paste listing1.1.txt listing1.2.txt

555-2397 Beckett, Barry 555-2397 unlisted
555-5116 Carter, Gertrude
555-5116 listed
555-7929 Jones, Theresa 555-7929 listed
555-9871 Orwell, Samuel 555-9871 unlisted
```

You can use paste to combine data from ﬁles that aren’t keyed with ﬁ elds suitable for use by join . Of course, to be meaningful, the ﬁles line numbers must be exactly equivalent. Alternatively, you can use paste as a quick way to create a two-column output of textual data; however, the alignment of the second column may not be exact if the ﬁrst
column’s line lengths aren’t exactly even.