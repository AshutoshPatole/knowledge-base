Join [[command]] :

The join command combines two ﬁles by matching the contents of speciﬁed ﬁelds within
the ﬁles. Fields are typically space-separated entries on a line, although you can specify
another character as the ﬁeld separator with the -t char option, where char is the character
you want to use. You can cause join to ignore case when performing comparisons by using
the -i option.

Listing 1.1: Demonstration File Containing Telephone Numbers and Names
555-2397        Beckett, Barry
555-5116        Carter, Gertrude
555-7929        Jones, Theresa
555-9871        Orwell, Samuel

Listing 1.2: Demonstration File Containing Telephone Number Listing Status
555-2397        unlisted
555-5116        listed
555-7929        listed
555-9871        unlisted


You can display the contents of both ﬁ les using join :

```
$ join listing1.1.txt listing1.2.txt

555-2397    Beckett, Barry          unlisted
555-5116    Carter, Gertrude      listed
555-7929    Jones, Theresa        listed
555-9871    Orwell, Samuel        unlisted
```

By default, join uses the ﬁ rst ﬁ eld as the one to match across ﬁ les. Because Listings 1.1 and
1.2 both place the phone number in this ﬁ eld, it’s the key ﬁ eld in the output. You can specify
another ﬁ eld by using the -1 or -2 option to specify the join ﬁ eld for the ﬁ rst or second ﬁ le,
respectively, as in join -1 3 -2 2 cameras.txt lenses.txt to join using the third ﬁ eld in
cameras.txt and the second ﬁ eld in lenses.txt .

The join command can be used at the core of a set of simple customized  database-manipulation tools using Linux text-manipulation commands. It’s very limited by itself, though; for instance, it requires its two ﬁ les to have the same ordering of lines.