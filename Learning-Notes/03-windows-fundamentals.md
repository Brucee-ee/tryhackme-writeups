# **Module 3: Windows and AD Fundamentals** 

_TryHackMe notes - Windows basics plus Active Directory_ 

This one's about Windows itself plus Active Directory (AD), which is basically how most companies manage all their Windows machines and user accounts centrally. AD comes up constantly in real pentests so worth properly understanding, not just skimming. 

## **Windows basics** 

- Registry - a giant database of settings for the OS and installed programs (regedit to view it) 

- Services - background processes that run without a user needing to be logged in (like a web server or antivirus) 

- Windows Updates / Windows Security / BitLocker - the built in tools for keeping a machine patched, protected, and encrypted 

## **What even is Active Directory** 

Basically imagine a company with 500 employees. Instead of managing 500 separate logins on 500 separate PCs, AD lets IT manage everyone's accounts, permissions, and PCs from one central place (a domain controller). 

- Domain - the whole managed network of users/computers 

- Domain Controller (DC) - the server running AD, the brain of the whole thing 

- Organisational Unit (OU) - a way of grouping users/computers (e.g. "Sales Dept", "IT Dept") to apply different rules to each group 

- Group Policy (GPO) - rules pushed out to users/computers automatically (e.g. "everyone in Sales gets this desktop background and can't install software") 

## **Authentication in AD** 

- Kerberos - the main way AD authenticates users these days, uses "tickets" instead of sending passwords around 

- NTLM - older authentication method, still around for compatibility but way less secure, big target for attackers 

Basically why AD matters for security: if you compromise the domain controller, you basically own the entire company network. Which is why AD attacks (like Kerberoasting, pass-the-hash etc, further down the line) are such a big deal in real pentests. 

