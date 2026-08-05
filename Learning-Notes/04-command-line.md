# **Module 4: Command Line** 

_TryHackMe notes - CMD, PowerShell, and Bash_ 

This module is basically drilling in the command line on both operating systems since GUI clicking around doesn't cut it in security work. Splits into Windows (CMD + PowerShell) and Linux (Bash). 

## **Windows CMD basics** 

dir            # list files (like ls) cd folder      # change directory ipconfig       # show network info tasklist       # show running processes whoami         # who am I logged in as 

## **PowerShell - the more powerful one** 

PowerShell is basically CMD's much stronger successor, everything in it is an object rather than just plain text, which makes it way more powerful for scripting. 

Get-Process          # list running processes Get-Service           # list services Get-ChildItem          # list files (PowerShell's version of ls/dir) Get-Help <command>     # PowerShell's built in help, actually really useful 

Cmdlets (that's what PowerShell commands are called) all follow a Verb-Noun pattern which honestly makes them easier to guess than you'd think once it clicks. 

## **Bash (Linux shell) basics** 

Covered a lot of this already in Linux Fundamentals, but this module goes a bit deeper into scripting concepts: 

- Piping | - send the output of one command into another, e.g. cat file.txt | grep 'error' 

- Redirection > and >> - send output into a file (> overwrites, >> appends) 

- Variables and basic if/loops for actual bash scripting (.sh files) 

## **Different types of shells** 

Worth knowing there's more than one type of shell (bash, sh, zsh, etc) and on the Windows side, CMD vs PowerShell vs WSL (which is basically real Linux running inside Windows). Which one you use often depends on what tools/scripts you're working with. 

