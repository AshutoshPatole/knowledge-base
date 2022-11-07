[[command]]
Sorting Files with sort :

Sometimes you’ll create an output ﬁle that you want sorted. To do so, you can use a command that’s called, appropriately enough, sort . This command can sort in several ways, including the following:
Ignore Case Ordinarily, sort sorts by ASCII value, which differentiates between uppercase and lowercase letters. The -f or --ignore-case option causes sort to ignore case. 

Month Sort The -M or --month-sort option causes the program to sort by three-letter month abbreviation ( JAN through DEC ).

Numeric Sort
You can sort by number by using the -n or --numeric-sort option.

Reverse Sort Order The -r or --reverse option sorts in reverse order.Processing Text Using Filters
23

Sort Field By default, sort uses the ﬁrst ﬁeld as its sort ﬁeld. You can specify another ﬁeld with the -k field or --key=field option. (The field can be two numbered ﬁelds separated
by commas, to sort on multiple ﬁelds.)

$ sort -k 3 listing1.1.txt

555-2397         Beckett, Barry
555-5116        Carter, Gertrude
555-9871         Orwell, Samuel
555-7929        Jones, Theresa

The sort command supports a large number of additional options, many of them quite exotic.