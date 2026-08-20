# **Module 9: Offensive Security Tooling**

_TryHackMe notes - Hydra, Gobuster, shells, and SQLMap_

This section was less about new concepts and more just building up an actual toolkit. Knowing tools exist is one thing, actually running them yourself is another.

## **Hydra**

Brute forcing tool, throws username/password combos at a login (SSH, FTP, web forms, loads of protocols supported).

```
hydra -l admin -P rockyou.txt ssh://10.10.10.10
```

Good reminder of why weak passwords and no lockout policies are still such a common finding in real assessments.

## **Gobuster: The Basics**

Directory/file enumeration on web servers, basically brute forcing URLs to find hidden pages or directories that aren't linked anywhere.

```
gobuster dir -u http://target.com -w wordlist.txt
```

Found this weirdly satisfying, watching a list of hidden paths pop up on something that looked totally empty at first glance.

## **Shells Overview**

The bit where reverse shells vs bind shells finally actually made sense instead of just being words I'd heard before.

- **Reverse shell** - target connects back out to you (usually gets past firewalls easier since outbound traffic is less restricted)
- **Bind shell** - target opens a port and waits for you to connect in

## **SQLMap: The Basics**

Automates SQL injection instead of doing it all manually like in the web hacking module. Point it at a URL and it'll test parameters for injection automatically.

```
sqlmap -u "http://target.com/page?id=1" --dbs
```

Felt a bit like cheating after doing SQLi by hand earlier, but good to understand both, manual for learning how it actually works, automated for real world speed.

## **Stuff worth remembering**

- These tools are loud, real assessments need permission and scope, not just firing them at random targets
- Wordlists matter as much as the tool itself, rockyou.txt comes up constantly
- Understanding the manual process first makes the automated tools way less of a black box
