## Lecture 2: Shell Tools and Scripting
Original note: [English Version](https://missing.csail.mit.edu/2020/course-shell/), [Chinese Version](https://missing-semester-cn.github.io/2020/shell-tools/)

Course video: [YouTube (Official)](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J), [Bilibili](https://www.bilibili.com/video/BV1x7411H7wa?p=1)
### Shell scripts
#### Variables
```
foo=bar
echo $foo
```
Note: `foo = bar` won't work. 
```
echo "$foo"
# bar
echo '$foo'
# $foo
```
Note: variables in `""` will be expanded, while that in `''` won't. 
#### .sh
```
vim mcd.sh
```
A quick note of `vim`: `i` to enter `INSERT` mode, `ESC` to get out of `INSERT` mode, and `:wq` to quit `vim`. 
```
# define mcd function
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```
```
source mcd.sh # load mcd function into the shell
mcd lecture2
pwd
# /mnt/e/test/lecture2
```
> - `$0` - Name of the script
> - `$1` to `$9` - Arguments to the script. `$1` is the first argument and so on.
> - `$@` - All the arguments
> - `$#` - Number of arguments
> - `$?` - Return error code of the previous command
> - `$$` - Process identification number (PID) for the current script
> - `!!` - Entire last command, including arguments. A common pattern is to execute a command only for it to fail due to missing permissions; you can quickly re-execute the command with `sudo` by doing `sudo !!`
> - `$_` - Last argument from the last command. If you are in an interactive shell, you can also quickly get this value by typing `Esc` followed by `.`
```
mkdir test
cd $_ # = cd test
echo "Hello"
echo $?
# 0
grep foobar mcd.sh  # search string "foobar" in mcd.sh
echo $?
# 1
true
echo $?
# 0
false
echo $?
# 1
```
#### conditions
`||` will execute the first one, and if the first one doesn't work, then execute the second one. 
```
false || echo "Oops fail"
# Oops fail
true || echo "Will not be printed"
#
```
`&&` will only execute the second one if the first one is run without errors. 
```
true && "Things went well"
# Things went well
false && echo "Will not be printed"
#
```
`;` concatenates commands in the same line. 
```
false; echo "This will always print"
# This will always print
```
#### Command substitution
```
echo "We are in $(pwd)"
# We are in /mnt/e/test/lecture2
cat <(ls) <(ls ..)  # write the result of a command to a temporary file and concat them
# lecture1
# lecture2
# mcd.sh
```
The `example.sh` is: 
```
#!/bin/bash

echo "Starting program at $(date)" # Date will be substituted

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # When pattern is not found, grep has exit status 1
    # We redirect STDOUT and STDERR to a null register since we do not care about them
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```
Note: `2>` ouputs standard error. `-ne` stands for not equal. 
```
./example.sh mcd.sh example.sh
# Starting program at Sun Jul 12 18:53:40 PDT 2020
# Running program ./example.sh with 2 arguments with pid 63
# File mcd.sh does not have any foobar, adding one
```
#### Globbing
```
ls *.sh
# example.sh  mcd.sh
ls lecture?  # expand to a single character
# lecture1:
# hello.txt   hello2.txt  'sample 4'   sample1   sample2   sample3.txt
# 
# lecture2:
# example.sh  mcd.sh
```
#### curly braces
```
convert image.png image.jpg
# or
convert image.{png,jpg}
```
```
touch foo.{,1,2,10}
```
