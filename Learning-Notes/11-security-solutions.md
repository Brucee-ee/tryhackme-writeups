# **Module 11: Security Solutions**

_TryHackMe notes - SIEM, firewalls, IDS, and vulnerability scanners_

This section was basically "what tools do blue teams actually use", putting names to concepts I'd heard thrown around loads of times before without really knowing what they meant.

## **Introduction to SIEM**

SIEM (Security Information and Event Management) pulls together logs from everywhere across a network into one place so analysts aren't checking a hundred different systems individually. Finally understood what people actually mean when they say SIEM instead of just nodding along like I knew.

## **Firewall Fundamentals**

Controls what traffic is allowed in/out of a network based on rules.

- **Stateless** - checks each packet on its own, no memory of previous traffic
- **Stateful** - tracks ongoing connections, smarter about what's legit traffic vs not

## **IDS Fundamentals**

Intrusion Detection System, monitors traffic/activity and flags anything suspicious.

- **Signature based** - matches against known attack patterns, fast but only catches what it already knows
- **Anomaly based** - flags anything that deviates from "normal" behaviour, can catch new stuff but more false positives

Worth noting IDS just detects, IPS (prevention) actually blocks it too.

## **Vulnerability Scanner Overview**

Automated tools that scan systems for known vulnerabilities and misconfigurations, basically doing at scale what I'd have to do manually otherwise. Good for getting a starting picture of a network's weak points before digging in deeper.

## **Stuff worth remembering**

- All of these tools generate more data than any human can look at manually, which is exactly why SIEMs and good alerting rules matter so much
- Anomaly based detection sounds cooler but signature based still catches the boring stuff that happens constantly
- Good to finally connect these tools to the SOC/DFIR stuff from the previous module, it's all the same ecosystem
