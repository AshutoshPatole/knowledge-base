[[command]]
|Test|Expression|
|---|---|
| -z STRING |	Empty string|
| -n STRING |	Not empty string|
| STRING == STRING |	Equal|
| STRING != STRING |	Not Equal|
| NUM -eq NUM |	Equal|
| NUM -ne NUM |	Not equal|
| NUM -lt NUM |	Less than|
| NUM -le NUM |	Less than or equal|
| NUM -gt NUM |	Greater than|
| NUM -ge NUM |	Greater than or equal|
| STRING =~ STRING |	Regexp|
|(( NUM < NUM ))|	Numeric conditions|
|-o noclobber |	If OPTIONNAME is enabled|
| ! EXPR |	Not|
| X && Y |	And|
| X (double pipe) Y |	Or|
| -e FILE | True if the FILE Exists|
| -r FILE |	True if the FILE is Readable|
| -h FILE |	True if the FILE is Symlink|
| -d FILE |	True if the FILE is Directory|
| -w FILE |	True if the FILE is Writable|
| -s FILE |	True if the FILE Size is > 0 bytes|
| -f FILE |	True if the FILE is a File|
| -x FILE |	True if the FILE is Executable|
| -b FILE | True if the FILE exists and is a block special file.|
| -c FILE | True if the FILE exists and is a special character file.|
| -G FILE | True if the FILE exists and has the same group as the user running the command.|
| -h FILE | True if the FILE exists and is a symbolic link.|
| -g FILE | True if the FILE exists and has set-group-id (sgid) flag set.|
| -k FILE | True if the FILE exists and has a sticky bit flag set.|
| -L FILE | True if the FILE exists and is a symbolic link.|
| -O FILE | True if the FILE exists and is owned by the user running the command.|
| -p FILE | True if the FILE exists and is a pipe.|
| -S FILE | True if the FILE exists and is socket.|
| -u FILE | True if the exists and set-user-id (suid) flag is set.|
| FILE1 -nt FILE2 |	The FILE 1 is more recent than FILE 2|
| FILE1 -ot FILE2  |	The FILE 2 is more recent than FILE 1|
| FILE1 -ef FILE2 |	Same files|

#### Note*
double pipe - `||` 

#shell #linux 