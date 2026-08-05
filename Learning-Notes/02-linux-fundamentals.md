# **Module 2: Linux Fundamentals** 

_TryHackMe notes - the big one, actually take this module seriously_ 

Big one. Linux is everywhere in security (most tools run on it, most servers run on it), so this module is worth actually taking seriously rather than rushing. 

## **Why Linux specifically** 

- Open source, so you can see exactly what it's doing (unlike Windows) 

- Most penetration testing distros (Kali, Parrot) are Linux based 

- A huge chunk of real-world servers run Linux, so if you're attacking or defending infrastructure you need to know it 

## **Filesystem basics** 

Linux doesn't have drive letters like C:\ - everything hangs off a single root, written as / 

- /home - user folders (like C:\Users on Windows) 

- /etc - config files live here 

- /var - variable stuff, logs mostly (/var/log is your best friend for troubleshooting) 

- /bin and /usr/bin - where executables/programs live 

- /root - the root user's home folder (not the same as /) 

## **Commands I actually use constantly** 

echo 'text'    # print text to the terminal whoami         # who am I logged in as pwd            # where am I (print working directory) ls -la         # list files, including hidden ones, with details cd folder/     # move into a folder cat file.txt   # print a file's contents grep 'word' file.txt   # search for text inside a file find / -name 'file.txt'   # search the whole filesystem for a file chmod 755 script.sh   # change file permissions sudo command   # run something as admin/root 

Also worth knowing the redirection/chaining operators from this room: > overwrites a file with output, >> appends to it instead of overwriting, && chains commands (only runs the second if the first succeeds), and & runs something in the background. 

## **Permissions (this confused me for ages)** 

When you run ls -la you get something like -rwxr-xr-- which breaks down as: 

- First char: file type (- = file, d = directory) 

- Next 3: owner's permissions (read/write/execute) 

- Next 3: group's permissions 

- Last 3: everyone else's permissions 

chmod 755 = owner gets read+write+execute (7), group and everyone else get read+execute (5 each). Numbers are just read=4, write=2, execute=1 added together. 

## **Users and sudo** 

root is the all-powerful admin account on Linux. Instead of logging in as root all the time (bad practice, too risky), normal users use sudo to run individual commands with elevated permissions when needed. 

