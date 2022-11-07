
Linux [[command]]
We often use $ everywhere in our scripts just to extract values from  a variable like
```bash
[thor@marvel-studios ~]$ echo $USER
thor
```
but it can be more than just this. Everything you pass to the $ is called a `parameter` and we can expand it with many ways like
- Converting to upper case
- Converting to lower case
- Slicing the string
- Remove specific parts from parameter value
- Search and Replace
- Finding length of the string
- Assigning default values
and so on and on.

These many things are possible with `${}` but we use it for just one purpose most of the time.

## Simple Use
The easiest way of using parameter name as we did in above snippet **`$USER`** but using it with braces (**`${USER}`**) has one advantage, it can be immediately followed by characters that would be interpreted as part of the parameter name otherwise.

```bash
[thor@marvel-studios ~]$ who="student"
[thor@marvel-studios ~]$ echo "$whos are playing in the garden"
 are playing in the garden
[thor@marvel-studios ~]$ echo "${who}s are playing in the garden"
students are playing in the garden
```
Also you have to use ${} for **positional arguments > $9**
```bash title="param_expansion.sh"
[thor@marvel-studios ~]$ cat param_expansion.sh 
#!/bin/bash

echo $1
echo $2
echo $3
echo $4 
echo $5
echo $6
echo $7
echo $8
echo $9
echo $10
echo $11

[thor@marvel-studios ~]$ ./param_expansion.sh one 2 three four 5 6 seven 8 9 ten eleven
one
2
three
four
5
6
seven
8
9
one0      # $10 is interpreted as $1 => one and 0 => one0
one1      # same for $11 => one1
```
Change the 10th and 11th line in param_expansion.sh file

```bash
echo ${10}
echo ${11}
```
and now it works as expected
```bash
[thor@marvel-studios ~]$ ./param_expansion.sh one 2 three four 5 6 seven 8 9 ten eleven
one
2
three
four
5
6
seven
8
9
ten
eleven
```

>[!note]
${} is parameter expansion and $() is **command substitution**. Both are entirely _different

## Finding length of the variable value
```bash
[thor@marvel-studios ~]$ who="thor"
[thor@marvel-studios ~]$ echo ${#who}
4    # t h o r
```

## Case modification
### Convert first letter to upper case ^

```bash
[thor@marvel-studios ~]$ who="thor"
[thor@marvel-studios ~]$ echo ${who}
thor
[thor@marvel-studios ~]$ echo ${who^}
Thor
```
### Convert first letter to lower case ,
```bash
[thor@marvel-studios ~]$ who="THOR"
[thor@marvel-studios ~]$ echo ${who}
THOR
[thor@marvel-studios ~]$ echo ${who,}
tHOR
```
### Convert all letter to upper case ^^

```bash
[thor@marvel-studios ~]$ who="thor"
[thor@marvel-studios ~]$ echo ${who}
thor
[thor@marvel-studios ~]$ echo ${who^^}
THOR
```
### Convert all letter to lower case ,,

```bash
[thor@marvel-studios ~]$ who="THOR"
[thor@marvel-studios ~]$ echo ${who}
THOR
[thor@marvel-studios ~]$ echo ${who,,}
thor
```

>[!tip]
A small script to change all files in current directory to lower case letters.
```bash
#!/bin/bash

for i in $(ls); do
        echo "Converting ${i} -> ${i,,}"
        mv ${i} ${i,,}
done
```

This script will throw warnings saying that files are similar if they are already lower case named. So tweak it.

## Search and Replace
Consider String as a parameter now which contains..
```bash
String="I am writing scripts. I am going to ececute it."
```
To replace a word **`{parameter/pattern/replaceString}`**
```bash
[thor@marvel-studios ~]$ echo ${String/I/You}
You am writing scripts. I am going to ececute it.
```
This only replaces the first occurenece. If you want to replace all occurences **`{parameter//pattern/replaceString}`**
```bash
[thor@marvel-studios ~]$ echo ${String//I/You}
You am writing scripts. You am going to ececute it.
```

## Slicing a String
Syntax **`{String:OffsetNumber}`**

```bash
[thor@marvel-studios ~]$ echo $String
I am writing scripts. I am going to ececute it.
[thor@marvel-studios ~]$ echo ${String:15}
ripts. I am going to ececute it.
[thor@marvel-studios ~]$ echo ${String:10}
ng scripts. I am going to ececute it.
```
You can also use length along with offset number
**`{String:OffsetNumber:length}`**

```bash
[thor@marvel-studios ~]$ echo $String
I am writing scripts. I am going to ececute it.
[thor@marvel-studios ~]$ echo ${String:10:5}
ng sc
[thor@marvel-studios ~]$ echo ${String:10:8}
ng scrip
[thor@marvel-studios ~]$ echo ${String:15:10}
ripts. I a
```

## Default values
### Use a default value
Syntax 
- **`${Parameter:-Default}`**
- **`${Parameter-Default}`**

If the Parameter is **unset** (never was defined) or **null** (empty), this one expands to Default, otherwise it expands to the value of Parameter, as if it just was ${Parameter}. If you **omit the :** (colon), like shown in the second form, the default value is only used when the Parameter was **unset**, not when it was empty.

```bash
echo "Your home directory is: ${HOME:-/home/$USER}."
echo "${HOME:-/home/$USER} will be used to store your personal data."
```
>[!note]
If HOME is unset or empty, everytime you want to print something useful, you need to put that parameter syntax in.


### Assign a default value

>[!caution]
Assign and Use of default values is entirely different. Read them carefully (**-** not equal to **=**).


Syntax
- **`${Parameter:=Default}`**
- **`${Parameter=Default}`**

This one works like the using default values, but the default text you give is not only expanded, but also assigned to the parameter, if it was **unset or null**. Equivalent to using a default value, when you omit the : (colon), as shown in the second form, the default value will only be assigned when the parameter was **unset**.
```bash
echo "Your home directory is: ${HOME:=/home/$USER}."
echo "$HOME will be used to store your personal data."
```
After the first expansion on line 1 `${HOME:=/home/$USER}`, HOME is **set and usable**.

#shell 