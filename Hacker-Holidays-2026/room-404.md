~~SS~~ 



<!-- Start of picture text -->
% rootitip-10-130-118-21 =<br>f up (8.9861 ency<br>he<br>' P/1.1 K<br>te: § Aug 2026 19:49:13 GMT<br>ntent-Length: 25<br>ache rol he<br><!-- End of picture text -->

Normally I'd have had to go hunting for hidden folders myself with gobuster or dirsearch, so having nmap just hand this to me was a nice shortcut. 

# Grabbing the repo 

Since the .git folder was just sitting there publicly, I didn't need to poke around file by file. I used a tool called git-dumper which basically rebuilds the whole repo for you from an exposed .git folder. 

```
pip3 install git-dumper
git-dumper http://10.130.171.197:8080/.git ./loot
```

That dumps everything into a folder called loot, and it comes out looking like a normal git repo I'd have cloned myself. 

# Digging through the history 

This was the bit that actually clicked for me doing this room. Normally when you look at a website you only see what's there right now. But because we've got the actual .git folder, we've got every past version of every file too. Even stuff that got deleted later is still sitting in the history somewhere. 

So instead of just opening the current files, I went and looked at the commit log: 

```
cd loot
git log --all
git log -p
```

git log --all just lists every commit that's ever been made. git log -p shows you the actual changes in each commit (the diff), which is what actually mattered here. 

Went back to the very first commit, called "initial Byte Lotus guest platform," and found this sitting in a diff on the README: 

```
+Staging flag(remove before launch): THM{byt3_l0tus_n3v3r_f0rg3ts}
```

So basically whoever built the site left themselves a little reminder note saying "remove this before going live," and then just... never did. And even if they had removed it later, it'd still be recoverable from the commit history anyway, which I guess is the whole point of the room. 

# Flag 

```
THM{byt3_l0tus_n3v3r_f0rg3ts}
```

# Stuff I took away from this one 

- Never let a .git folder end up on a live web server. Should only ever exist on your own machine or somewhere private like GitHub, not sitting next to the actual site files. 

- Deleting something doesn't actually delete it from git history. If you accidentally commit a password or key, just removing it in a later commit isn't enough, it's still recoverable. You'd need to actually rotate whatever got leaked and then scrub it out of history properly (tools like git filter-repo exist for that). 

- Gonna remember nmap's http-git script for future rooms, saved me a whole step of manual dirbusting. 

# Tools I used 

- nmap (the http-git script specifically) 

- git-dumper 

- git log / git log -p 

