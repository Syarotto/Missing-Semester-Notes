## Lecture 2: Shell Tools and Scripting
Original note: [English Version](https://missing.csail.mit.edu/2020/shell-tools/), [Chinese Version](https://missing-semester-cn.github.io/2020/shell-tools/)

Course video: [YouTube (Official)](https://youtu.be/kgII-YWo3Zw), [Bilibili](https://www.bilibili.com/video/BV1x7411H7wa?p=2)
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
touch lecture{1,2}/test{1,2,3}.py   # Cartesian product
msdir foo bar
touch {foo,bar}/{a..j}
touch foo/x foo/y
diff <(ls foo) <(ls bar)
# 11,12d10
# < x
# < y
```
#### Shebang
An example in `python`:
```
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```
To run it, 
```
./script.py a b c
# c
# b
# a
```
Note: `#!/usr/local/bin/python` is the reason why the shell knows that this is a python script. Another way is to include `#!/usr/bin/env python`, which will use the `python` program found in `$PATH` to interpret the script. 
#### Shellcheck
```
shellcheck mcd.sh
# In mcd.sh line 1:
# # foobar
# ^-- SC2148: Tips depend on target shell and yours is unknown. Add a shebang.
```
### Shell tools
#### Tldr
Provides commonly used examples of commands. 
```
tldr tar
```
#### Find
Recursively goes through all folders under the given directory and find files matching the pattern. 
```
find . -name foo -type d
find . -path '**/test/**/*.py' -type f
find . -mtime -1    # modify time
find . -size +500k -size -10M -name '*.tar.gz'
```
It can also execute certain commands after finding all desired files. 
```
find . -name '*.tmp' -exec rm {} \;
find . -name '*.png' -exec convert {} {.}.jpg \;
```
An alternative command for `find` is `fd`. [Here](https://github.com/sharkdp/fd) is the installation instruction. *(Do not use the provided `sudo apt install fdclone` in the shell to install `fd`, as it's a different library. )* `fd` use [regular expression](https://en.wikipedia.org/wiki/Regular_expression) instead of globbing to match patterns. A website I found very useful to test regex is [RegExr](https://regexr.com/), which not only highlights matched patterns but also includes explanation of every regex. 
```
fd ".*py" 
```
#### Locate
Searches for files efficiently. 
```
locate tmp
sudo updatedb
```
Note: it seems that `locate` can only find files within the linux file space. For more information, visit [locate and updatedb - Issue](https://github.com/Microsoft/WSL/issues/369). On Windows, [Everything](https://en.wikipedia.org/wiki/Everything_(software)) is a fairly handy software to search for files efficiently. 
#### Grep
Finds patterns within the file. 
```
grep foobar mcd.sh
grep -R foobar .    # search in all files under the given directory
```
An alternative of `grep` is `ripgrep`, which can be installed via [instruction](https://github.com/BurntSushi/ripgrep#installation). It has nicely formatted results with more information. 
```
rg "for" -t py .
# ./script.py
# 3:for arg in reversed(sys.argv[1:]):
rg "for" -t py -C 5 .   # -C gives context
# ./script.py
# 1-#!/usr/local/bin/python
# 2-import sys
# 3:for arg in reversed(sys.argv[1:]):
# 4-        print(arg)
rg -u --files-without-match "^#!" -t sh    # -u doesn't ignore hidden files
```
Unlike the example given in the lecture, either `rg -u --files-without-match "^#!" -t sh` or `rg -u --files-without-match ^#\! -t sh` works, while `rg -u --files-without-match "^#\!" -t sh` throws out an erroe saying `error: unrecognized escape sequence`. Recall in the last lecture that we have `echo "Hello world"` or `echo Hello\ world`, my guess is that we don't need escape sequence if it's quoted. I don't know, however, why it's working in the lecture. 
```
rg -u --files-without-match ^#\! -t sh --stats
# mcd.sh
# 
# 1 matches
# 1 matched lines
# 1 files contained matches
# 2 files searched
# 0 bytes printed
# 531 bytes searched
# 0.000342 seconds spent searching
# 0.007829 seconds
```
#### History
Outputs command history. 
```
history | grep convert  # find all commands that have convert
```
`Ctrl` + `R` provides reversed history search. 
#### Fzf
Interactive search. Installtion documentation can be found [here](https://github.com/junegunn/fzf#using-git). 
```
cat example.sh | fzf
```
If default bindings is chosen during installation, then now `Ctrl` + `R` will have a similar nicely formatted search window. It also supports fuzzy search, which means you don't have to type in regex like `find`. 
#### Zsh
To set up `zsh` on WSL, I followed this [blog](https://blog.joaograssi.com/windows-subsystem-for-linux-with-oh-my-zsh-conemu/). A Chinese instruction on [zhihu](https://zhuanlan.zhihu.com/p/62501175) might also help. 

Note: During installation, I encountered a problem. After running `chsh -s /bin/zsh` and rebooting the system, `echo $SHELL` still gets `/bin/bash`. Several posts ([1](https://askubuntu.com/questions/195361/chsh-s-usr-bin-zsh-not-working), [2](https://askubuntu.com/questions/119253/ive-changed-default-shell-but-my-terminal-dont-get-it)) didn't help, and since everything else went well, I decided to ignore this issue for now. 

The auto suggestion plugin mentioned in the lecture is `zsh-autosuggestions`. 
#### Tree
Displays the tree structure under current directory. 
```
tree
```
Note: `blue` means it's a directory, and `red` means it has execuation permission. 
An alternative of it is `broot`, but I got an error when running `cargo install broot`, which might be related to this [issue](https://github.com/rust-lang/rust/pull/68658). 
## Exercises
- Read `man ls` and write an `ls` command that lists files in the following manner
    - Includes all files, including hidden files
    - Sizes are listed in human readable format (e.g. 454M instead of 454279954)
    - Files are ordered by recency
    - Output is colorized
---
```
ls -l -a -h -t --color=auto
```
---
- Write bash functions `marco` and `polo` that do the following. Whenever you execute `marco` the current working directory should be saved in some manner, then when you execute `polo`, no matter what directory you are in, `polo` should `cd` you back to the directory where you executed `marco`. For ease of debugging you can write the code in a file `marco.sh` and (re)load the definitions to your shell by executing `source marco.sh`.
---
---
- Say you have a command that fails rarely. In order to debug it you need to capture its output but it can be time consuming to get a failure run. Write a bash script that runs the following script until it fails and captures its standard output and error streams to files and prints everything at the end. Bonus points if you can also report how many runs it took for the script to fail.
```
 #!/usr/bin/env bash
 n=$(( RANDOM % 100 ))
 if [[ n -eq 42 ]]; then
    echo "Something went wrong"
    >&2 echo "The error was using magic numbers"
    exit 1
 fi
 echo "Everything went according to plan"
```
---
The `detect.sh` script is: 
```
#!/usr/bin/env bash
count=0
until [[ $STATUS -eq 1 ]]
do
    ./error.sh
    STATUS=$?
    count=`expr $count + 1`
done
echo "Runs $count times in total"
```
Note: 
1. `$?` refers to the exact command before it, so if adding a counter and still use `$?` in the condition, the program will not end; 
2. remember to append white spaces around the condition in `[[]]`;
3. there are various ways to increase a variable in bash; see [here](https://askubuntu.com/questions/385528/how-to-increment-a-variable-in-bash). Please note that it's backquote instead of single quote in the one I used. 

The command to run the script: 
```
./detect.sh
```
---
