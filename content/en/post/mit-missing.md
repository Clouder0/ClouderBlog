---
title: "MIT Missing Semester Exercises"
date: 2022-08-21T22:57:10+08:00
slug: "mit-missing"
type: post
tags:
  - linux
  - shell
categories:
  - Programming
---

* **1/13/20** : [Course overview + the shell](https://missing.csail.mit.edu/2020/course-shell/)
* **1/14/20** : [Shell Tools and Scripting](https://missing.csail.mit.edu/2020/shell-tools/)
* **1/15/20** : [Editors (Vim)](https://missing.csail.mit.edu/2020/editors/)
* **1/16/20** : [Data Wrangling](https://missing.csail.mit.edu/2020/data-wrangling/)
* **1/21/20** : [Command-line Environment](https://missing.csail.mit.edu/2020/command-line/)
* **1/22/20** : [Version Control (Git)](https://missing.csail.mit.edu/2020/version-control/)
* **1/23/20** : [Debugging and Profiling](https://missing.csail.mit.edu/2020/debugging-profiling/)
* **1/27/20** : [Metaprogramming](https://missing.csail.mit.edu/2020/metaprogramming/)
* **1/28/20** : [Security and Cryptography](https://missing.csail.mit.edu/2020/security/)
* **1/29/20** : [Potpourri](https://missing.csail.mit.edu/2020/potpourri/)

## Lesson 1

1. For this course, you need to be using a Unix shell like Bash or ZSH. If you are on Linux or macOS, you don’t have to do anything special. If you are on Windows, you need to make sure you are not running cmd.exe or PowerShell; you can use [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) or a Linux virtual machine to use Unix-style command-line tools. To make sure you’re running an appropriate shell, you can try the command `echo $SHELL`. If it says something like `/bin/bash` or `/usr/bin/zsh`, that means you’re running the right program.
2. Create a new directory called `missing` under `/tmp`.

    ```bash
    cd /tmp
    mkdir missing
    ```
3. Look up the `touch` program. The `man` program is your friend.

    ```bash
    man touch
    ```
4. Use `touch` to create a new file called `semester` in `missing`.

    ```bash
    cd /tmp/missing
    touch semester
    ```
5. Write the following into that file, one line at a time:

    ```
    #!/bin/sh
    curl --head --silent https://missing.csail.mit.edu
    ```

    The first line might be tricky to get working. It’s helpful to know that `#` starts a comment in Bash, and `!` has a special meaning even within double-quoted (`"`) strings. Bash treats single-quoted strings (`'`) differently: they will do the trick in this case. See the Bash [quoting](https://www.gnu.org/software/bash/manual/html_node/Quoting.html) manual page for more information.

    ```bash
    cat '#!/bin/sh' >> semester
    cat 'curl --head --silent https://missing.csail.mit.edu' >> semester
    ```
6. Try to execute the file, i.e. type the path to the script (`./semester`) into your shell and press enter. Understand why it doesn’t work by consulting the output of `ls` (hint: look at the permission bits of the file).

    ```bash
    ./semester
    ls -l
    ```

    The file semester has no execution permission.
7. Run the command by explicitly starting the `sh` interpreter, and giving it the file `semester` as the first argument, i.e. `sh semester`. Why does this work, while `./semester` didn’t?
8. Look up the `chmod` program (e.g. use `man chmod`).
9. Use `chmod` to make it possible to run the command `./semester` rather than having to type `sh semester`. How does your shell know that the file is supposed to be interpreted using `sh`? See this page on the [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) line for more information.

    ```bash
    chmod +x semester
    ```
10. Use `|` and `>` to write the “last modified” date output by `semester` into a file called `last-modified.txt` in your home directory.

     ```bash
     date -r test | cat > ~/last-modified.txt
     ```
11. Write a command that reads out your laptop battery’s power level or your desktop machine’s CPU temperature from `/sys`. Note: if you’re a macOS user, your OS doesn’t have sysfs, so you can skip this exercise.

     ```bash
     cat /sys/class/power_supply/BAT0/capacity
     ```

## Lesson 2

1. Read [`man ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html) and write an `ls` command that lists files in the following manner

    * Includes all files, including hidden files
    * Sizes are listed in human readable format (e.g. 454M instead of 454279954)
    * Files are ordered by recency
    * Output is colorized

    A sample output would look like this

    ```
     -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
     drwxr-xr-x   5 user group  160 Jan 14 09:53 .
     -rw-r--r--   1 user group  514 Jan 14 06:42 bar
     -rw-r--r--   1 user group 106M Jan 13 12:12 foo
     drwx------+ 47 user group 1.5K Jan 12 18:08 ..
    ```

    Sol:

    ```bash
    ls -lhat --color=auto
    ```
2. Write bash functions `marco` and `polo` that do the following. Whenever you execute `marco` the current working directory should be saved in some manner, then when you execute `polo`, no matter what directory you are in, `polo` should `cd` you back to the directory where you executed `marco`. For ease of debugging you can write the code in a file `marco.sh` and (re)load the definitions to your shell by executing `source marco.sh`.

    ```bash
    marco()
    {
    	cur=$(pwd)
    }
    polo()
    {
    	cd $cur
    }
    ```
3. Say you have a command that fails rarely. In order to debug it you need to capture its output but it can be time consuming to get a failure run. Write a bash script that runs the following script until it fails and captures its standard output and error streams to files and prints everything at the end. Bonus points if you can also report how many runs it took for the script to fail.

    ```bash
     #!/usr/bin/env bash

     n=$(( RANDOM % 100 ))

     if [[ n -eq 42 ]]; then
        echo "Something went wrong"
        >&2 echo "The error was using magic numbers"
        exit 1
     fi

     echo "Everything went according to plan"
    ```

    Sol:

    ```bash
    #!/usr/bin/bash

    n=0
    ./run.sh >/dev/null
    while [[ $? -ne 1 ]];
    do
        n=$(( $n + 1 ))
        echo $n
        ./run.sh >/dev/null
    done
    echo "Error!"
    ```
4. As we covered in the lecture `find`’s `-exec` can be very powerful for performing operations over the files we are searching for. However, what if we want to do something with **all** the files, like creating a zip file? As you have seen so far commands will take input from both arguments and STDIN. When piping commands, we are connecting STDOUT to STDIN, but some commands like `tar` take inputs from arguments. To bridge this disconnect there’s the [`xargs`](https://www.man7.org/linux/man-pages/man1/xargs.1.html) command which will execute a command using STDIN as arguments. For example `ls | xargs rm` will delete the files in the current directory.  
    Your task is to write a command that recursively finds all HTML files in the folder and makes a zip with them. Note that your command should work even if the files have spaces (hint: check `-d` flag for `xargs`).  
    If you’re on macOS, note that the default BSD `find` is different from the one included in [GNU coreutils](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands). You can use `-print0` on `find` and the `-0` flag on `xargs`. As a macOS user, you should be aware that command-line utilities shipped with macOS may differ from the GNU counterparts; you can install the GNU versions if you like by [using brew](https://formulae.brew.sh/formula/coreutils).

    ```bash
    find . -name "*.html" | xargs -d "\n" tar cf target.tar
    ```
5. (Advanced) Write a command or script to recursively find the most recently modified file in a directory. More generally, can you list all files by recency?

    ```bash
    tac <(find . -exec sh -c "date -r {} | xargs echo -n; echo -n '|'; echo -n {}; echo ;" \; | sort)
    ```

## Lesson 5

## Job control

1. From what we have seen, we can use some `ps aux | grep` commands to get our jobs’ pids and then kill them, but there are better ways to do it. Start a `sleep 10000` job in a terminal, background it with `Ctrl-Z` and continue its execution with `bg`. Now use [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) to find its pid and [`pkill`](http://man7.org/linux/man-pages/man1/pgrep.1.html) to kill it without ever typing the pid itself. (Hint: use the `-af` flags).

    ```bash
    sleep 10000 &
    pgrep sleep | kill
    ```
2. Say you don’t want to start a process until another completes. How would you go about it? In this exercise, our limiting process will always be `sleep 60 &`. One way to achieve this is to use the [`wait`](https://www.man7.org/linux/man-pages/man1/wait.1p.html) command. Try launching the sleep command and having an `ls` wait until the background process finishes.  
    However, this strategy will fail if we start in a different bash session, since `wait` only works for child processes. One feature we did not discuss in the notes is that the `kill` command’s exit status will be zero on success and nonzero otherwise. `kill -0` does not send a signal but will give a nonzero exit status if the process does not exist. Write a bash function called `pidwait` that takes a pid and waits until the given process completes. You should use `sleep` to avoid wasting CPU unnecessarily.

    ```bash
    #!/usr/bin/bash
    pidwait()
    {
        kill -0 $1
        while [[ $? -ne 1 ]];
        do
            sleep 1
            kill -0 $1
        done
        echo Done.
    }
    ```

## Terminal multiplexer

1. Follow this `tmux` [tutorial](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) and then learn how to do some basic customizations following [these steps](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/).

## Aliases

1. Create an alias `dc` that resolves to `cd` for when you type it wrongly.

    ```bash
    alias dc=cd
    ```
2. Run `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10` to get your top 10 most used commands and consider writing shorter aliases for them. Note: this works for Bash; if you’re using ZSH, use `history 1` instead of just `history`.

    ```bash
    history | sed -E 's/\s+[0-9]+\s+//' | sort | uniq -c | sort -n | tail -n 10
    ```

## Dotfiles

Let’s get you up to speed with dotfiles.

1. Create a folder for your dotfiles and set up version control.
2. Add a configuration for at least one program, e.g. your shell, with some customization (to start off, it can be something as simple as customizing your shell prompt by setting `$PS1`).
3. Set up a method to install your dotfiles quickly (and without manual effort) on a new machine. This can be as simple as a shell script that calls `ln -s` for each file, or you could use a [specialized utility](https://dotfiles.github.io/utilities/).
4. Test your installation script on a fresh virtual machine.
5. Migrate all of your current tool configurations to your dotfiles repository.
6. Publish your dotfiles on GitHub.

> I would recommend `yadm` for managing dotfiles.
>

## Lesson 6

1. If you don’t have any past experience with Git, either try reading the first couple chapters of [Pro Git](https://git-scm.com/book/en/v2) or go through a tutorial like [Learn Git Branching](https://learngitbranching.js.org/). As you’re working through it, relate Git commands to the data model.
2. Clone the [repository for the class website](https://github.com/missing-semester/missing-semester).

    1. Explore the version history by visualizing it as a graph.

        ```bash
        git log --graph
        ```
    2. Who was the last person to modify `README.md`? (Hint: use `git log` with an argument).

        ```bash
        git log --raw
        ```
    3. What was the commit message associated with the last modification to the `collections:` line of `_config.yml`? (Hint: use `git blame` and `git show`).

        ```bash
        git blame _config.yml -L 18
        git show a88b4eac
        ```
3. One common mistake when learning Git is to commit large files that should not be managed by Git or adding sensitive information. Try adding a file to a repository, making some commits and then deleting that file from history (you may want to look at [this](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)).
4. Clone some repository from GitHub, and modify one of its existing files. What happens when you do `git stash`? What do you see when running `git log --all --oneline`? Run `git stash pop` to undo what you did with `git stash`. In what scenario might this be useful?
5. Like many command line tools, Git provides a configuration file (or dotfile) called `~/.gitconfig`. Create an alias in `~/.gitconfig` so that when you run `git graph`, you get the output of `git log --all --graph --decorate --oneline`. Information about git aliases can be found [here](https://git-scm.com/docs/git-config#Documentation/git-config.txt-alias).

    ```bash
    [alias]
        graph = log --all --graph --decorate --oneline
    ```
6. You can define global ignore patterns in `~/.gitignore_global` after running `git config --global core.excludesfile ~/.gitignore_global`. Do this, and set up your global gitignore file to ignore OS-specific or editor-specific temporary files, like `.DS_Store`.
7. Fork the [repository for the class website](https://github.com/missing-semester/missing-semester), find a typo or some other improvement you can make, and submit a pull request on GitHub (you may want to look at [this](https://github.com/firstcontributions/first-contributions)).

---