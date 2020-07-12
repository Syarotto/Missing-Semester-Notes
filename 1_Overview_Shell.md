## Lecture 1: Course Overview + The Shell
Original note: [English Version](https://missing.csail.mit.edu/2020/course-shell/), [Chinese Version](https://missing-semester-cn.github.io/2020/course-shell/)

Course video: [YouTube (Official)](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J), [Bilibili](https://www.bilibili.com/video/BV1x7411H7wa?p=1)
### Date
Presents the date and the time. 
```
date
# Saturday, July 11, 2020 3:49:44 PM
```
### Echo
Prints out the argument. 
```
echo hello
# hello
```
Note: to print *Hello world*, use `echo "Hello world"` or `echo Hello\ world`. 
```
echo $PATH  # all paths that the shell will search for programs
which echo  # if I were to run a program called echo, I would run this one
# /bin/echo
```
Note: on Linux or MacOS, `/` is the root path, while on Windows, there is one root for each partition, such as `C:\` and `D:\`. 
### Directory
#### pwd (print working directory)
Prints the current directory I'm in. 
```
pwd
# /mnt/c/Users/Charlotte
```
#### cd (change directory)
```
cd /mnt/e
pwd
# /mnt/e
```
Note: `.` means the current directory and `..` means the parent directory. 
*To directly run a program from anywhere, either the shell could find where it is using PATH, or an absolute path is given. *
#### ls (list)
Lists all files under the current directory. 
```
pwd
# /mnt/e/test/lecture1
ls
# sample1  sample2
ls ..
# lecture1
```
Note: `~` is expanded to the home directory.
```
cd ~
pwd
# /home/syarotto
```
Note: `-` is expanded to the previous directory. 
```
cd -
# /mnt/e/test/lecture1
```
#### flags and options
```
ls --help
# Usage: ls [OPTION]... [FILE]...
# List information about the FILEs (the current directory by default).
# Sort entries alphabetically if none of -cftuvSUX nor --sort is specified.
# 
# Mandatory arguments to long options are mandatory for short options too.
#   -a, --all                  do not ignore entries starting with .
#   -A, --almost-all           do not list implied . and ..
# ...
```
Note: `-l` gives a long listing format. 
```
ls -l
# total 0
# drwxrwxrwx 1 syarotto syarotto 512 Jul 11 16:21 sample1
# drwxrwxrwx 1 syarotto syarotto 512 Jul 11 16:22 sample2
# -rwxrwxrwx 1 syarotto syarotto   0 Jul 11 16:34 sample3.txt
```
For the first column, `d` stands for a directory, and the rest are permissions. The first group of 
three are permissions set for the user owner of the file. The second group of three are permissions 
set for the group that owns this file. The last group of three are permissions set for everyone else. 
Inside the group of three, they are read, write and execute. For directories, `r` means you are allowed 
to see which files are inside the directory, `w` means you are allowed to rename, create or remove files 
within the directory, and `x` means you are allowed to enter the directory. 
#### mv (move)
Moves and renames a file. 
```
 mv sample3.txt ../sample.txt # mv <original file> <target file>
 ls ..
 # lecture1  sample.txt
```
#### cp (copy)
Copies and renames a file. 
```
cp sample.txt ./lecture1/sample3.txt
ls ./lecture1
# sample1  sample2  sample3.txt
```
#### rm (remove)
Removes a file. 
```
rm sample.txt
ls
# lecture1
```
Note: `rm` on Linux is by default not recursive. Thus, to remove a directory and everything inside it, use `rm -r <directory>`, 
where `-r` stands for recursively. `rmdir` could also removes a directory, but only when the directory is empty. 
#### mkdir (make directory)
Makes a new directory. 
```
mkdir "sample 4"
ls
# 'sample 4'   sample1   sample2   sample3.txt
```
### Man (manual pages)
Takes the name of another program and gives its manual page. 
```
man ls
```
Note: `Ctrl`+ `L` can clear the terminal. 
### Streams
#### < and >
`< file` means to rewire the input of the program to be the content of the file, and `> file` means to rewire the output of 
the proceeding program into this file. 
```
echo hello > hello.txt
cat hello.txt # prints the content of a file
# hello
cat < hello.txt
# hello
cat < hello.txt > hello2.txt  # takes the content of hello.txt as its input and write it to hello2.txt
cat hello2.txt
hello
```
`>>` means to append content to an existing file. 
```
cat < hello.txt >> hello2.txt
cat hello2.txt
# hello
# hello
```
#### | (pipe)
Takes the ouput of the program to the left and makes it the input of the program to the right. 
```
ls -l | tail -n1  # tail gives the last n lines of the input
# -rwxrwxrwx 1 syarotto syarotto   0 Jul 11 16:47 sample3.txt
```
### sudo (do as super user)
Runs the command as if you were the root. 

*The following commands seem not working neither in WSL nor in Powershell because there is no such directory.*
```
cd /sys/class/backlight/intel_backlight
sudo su # run the following commands as the root,  and $ will be changed to #. 
echo 500 > brightness
```
Or: 
```
echo 1060 | sudo tee brightness # tee takes the input and writes it to a file and a stdout
```
### xdg-open
Opens a file using appropriate program. 
```
xdg-open sample3.txt
```
*(`less` should also work. )*
## Exercises
- Create a new directory called `missing` under `/tmp`.
```
# Answer
mkdir /tmp/missing
```
- Look up the `touch` program. The `man` program is your friend.
```
# Answer
man touch
```
- Use touch to create a new file called `semester` in `missing`. 
```
# Answer
touch missing/semester
```
- Write the following into that file, one line at a time:
```
#!/bin/sh
curl --head --silent https://missing.csail.mit.edu
```
```
# Answer
echo '#!/bin/sh' >> semester
echo "curl --head --silent https://missing.csail.mit.edu" >> semester
# or
echo "\#\!/bin/sh" >> semester
echo "curl --head --silent https://missing.csail.mit.edu" >> semester
```
*Reference: what is [Bash history expansion](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#History-Interaction)
and [how to get rid of it](https://stackoverflow.com/a/11816138)*
- Try to execute the file, i.e. type the path to the script (`./semester`) into your shell and press enter. 
Understand why it doesn’t work by consulting the output of `ls` (hint: look at the permission bits of the file).
```
./semester
# -bash: ./semester: Permission denied
ls -l
# total 0
# -rw-rw-rw- 1 syarotto syarotto 61 Jul 11 18:21 semester
```
Note that we don't have the execute permission of the file. 
- Run the command by explicitly starting the `sh` interpreter, and giving it the file `semester` as the first argument,
i.e. `sh semester`. Why does this work, while `./semester` didn’t?
 ```
 # Answer
 sh ./semester
 ```
Because if running `sh`, we only need read access to the file. 

*Reference: [Run ./script.sh vs bash script.sh - permission denied](https://unix.stackexchange.com/questions/203371/run-script-sh-vs-bash-script-sh-permission-denied)*

Extended notes: 
1. [Difference between sh and bash](https://stackoverflow.com/questions/5725296/difference-between-sh-and-bash)
2. [Curl](https://www.geeksforgeeks.org/curl-command-in-linux-with-examples/)
- Look up the `chmod` program (e.g. use `man chmod`).
```
# Answer
man chmod
```
- Use `chmod` to make it possible to run the command `./semester` rather than having to type `sh semester`. 
How does your shell know that the file is supposed to be interpreted using `sh`? 
```
# Answer
chmod u=rwx,g=rwx,o=rwx semester
ls -l
# total 0
# -rwxrwxrwx 1 syarotto syarotto 61 Jul 11 18:21 semester
./semester
```
Because the first line in the file `#!/bin/sh` specified the interpreter program. 

*Reference: [Shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))*
- Use `|` and `>` to write the "last modified" date output by `semester` into a file called `last-modified.txt` in your home directory.
```
# Answer
date -r semester > last-modified.txt | mv last-modified.txt ~/last-modified.txt
```
