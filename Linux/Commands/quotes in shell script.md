
Linux [[command]]
Quotes in bash script are confusing. I often get confused why my script is not behaving properly even if all the commands are correct and most of the times the culprit are "quotes" surrounding the command.
Let's understand them with an example.

```bash
[thor@marvel-studios ~]$ echo Bring Mjollnir     near me.
Bring Mjollnir near me.
```
but if I quote them 
```bash
[thor@marvel-studios ~]$ echo "Bring Mjollnir     near me."
Bring Mjollnir     near me.
```
It works. Till now you might have guessed it right but things become weird when you store the result in a variable

```bash
[thor@marvel-studios ~]$ infinity="Bring Mjollnir     near me."
[thor@marvel-studios ~]$ echo $infinity
Bring Mjollnir near me.
```

Did you noticed ? We surrounded the sentence with double quotes and saved in `infinity` variable but outputting the variable didn't worked as expected but what if we quote the variable also ?

```bash
[thor@marvel-studios ~]$ echo "$infinity"
Bring Mjollnir     near me.
```
![confused GIF](https://c.tenor.com/2f31yo7eV6kAAAAC/confused-confusion.gif)

Okay let' clear the confusion. In bash scripts there are often times when we want to escape characters which can be done by using `\ (forward slash)` or through quotes either single `'` or double `"` but these quotes are totally different from each other. Let's take an example

```bash
[thor@marvel-studios ~]$ echo 'Bring Mjollnir     near me.'
Bring Mjollnir     near me.
```
We got the same output as when we used `"` but now let's use the variable now

```bash
[thor@marvel-studios ~]$ sentence="Bring Mjollnir     near me."
[thor@marvel-studios ~]$ echo '$sentence'
$sentence
```
Ooh-la-la we got what we typed. If you use `' '` single quotes every special character
- $
- /
- (asterisk) *
- &
- !
- blank space … etc.,
which are frequently used in bash scripts are ignored which means what you type is what you get. Whereas if you use `" "` double quotes only `"blank   spaces"` are ignored and the other special characters mentioned above retain there functionality this is why it results like this.
```bash
[thor@marvel-studios ~]$ echo '$sentence'
$sentence
[thor@marvel-studios ~]$ echo "$sentence"
Bring Mjollnir     near me.
```

Double quotes also helps in keeping the format of the output perfect.

```bash
[thor@marvel-studios ~]$ echo $(ls -la)
total 28 drwx------. 4 thor thor 142 Feb 7 04:35 . drwxr-xr-x. 4 root root 29 Feb 4 07:27 .. -rwx------. 1 thor thor 726 Feb 6 14:49 .bash_history -rwx------. 1 thor thor 18 Oct 10 01:59 .bash_logout -rwx------. 1 thor thor 141 Oct 10 01:59 .bash_profile -rwx------. 1 thor thor 563 Feb 4 09:19 .bashrc drwxrwxr-x. 3 thor thor 20 Feb 7 04:35 hello -rwxrwxr-x. 1 thor thor 71 Feb 7 04:35 sample.sh -rw-------. 1 thor thor 7321 Feb 7 04:35 .viminfo drwxrwxr-x. 3 thor thor 20 Feb 7 04:35 world



[thor@marvel-studios ~]$ echo "$(ls -la)"
total 28
drwx------. 4 thor thor  142 Feb  7 04:35 .
drwxr-xr-x. 4 root root   29 Feb  4 07:27 ..
-rwx------. 1 thor thor  726 Feb  6 14:49 .bash_history
-rwx------. 1 thor thor   18 Oct 10 01:59 .bash_logout
-rwx------. 1 thor thor  141 Oct 10 01:59 .bash_profile
-rwx------. 1 thor thor  563 Feb  4 09:19 .bashrc
drwxrwxr-x. 3 thor thor   20 Feb  7 04:35 hello
-rwxrwxr-x. 1 thor thor   71 Feb  7 04:35 sample.sh
-rw-------. 1 thor thor 7321 Feb  7 04:35 .viminfo
drwxrwxr-x. 3 thor thor   20 Feb  7 04:35 world
```


> [!tip]
Always use **`" "`** double quotes because they are easy to differentiate. single quotes ' and backtick ` are kind of hard to distinguish when you are debugging fast. And if you ever want to get ignore some special character like $ in " " you can use \ as
	```bash
	[thor@marvel-studios ~]$ echo "\$sentence"
	$sentence
	```
	

> [!tip]
> **The golden rule on quoting is very simple:**
>  _If there is whitespace or a symbol in your argument, you must quote it._
> If there isn't, quotes are usually optional, but you can still quote it to be safe._

It is extremely rare for arguments to require no quoting, primarily inside tests and around ${..+..} expansions. Do not remove or omit quotes from your arguments to try and make something work in any other case; you are much more likely to introduce a terrible and difficult to detect bug instead.

Always use **`" "`** double quotes because they are easy to differentiate than single quotes ' and backtick `.



#shell 