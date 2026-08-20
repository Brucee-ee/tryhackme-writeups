# **Module 12: Defensive Security Tooling**

_TryHackMe notes - CyberChef, CAPA, REMnux, and FlareVM_

Last stretch of the path and a good one to end on, very hands on, actual tools you'd use in a real DFIR case rather than just theory.

## **CyberChef: The Basics**

Already loved this one from other projects before even doing this module properly. Basically a drag and drop tool for decoding, encoding, and transforming data, Base64, hex, XOR, hashing, all in a visual pipeline instead of writing scripts for every single thing.

- Handy for CTFs when you're not sure what encoding something is, just try recipes until it looks readable
- Genuinely one of the tools I reach for the most already

## **CAPA: The Basics**

Malware analysis tool that looks at a binary and tells you what capabilities it has (network access, file writing, persistence mechanisms etc) without having to fully reverse engineer it by hand first. Good starting point before diving deeper into a sample.

## **REMnux: Getting Started**

A whole Linux distro built specifically for malware analysis, comes preloaded with a load of the tools you'd need instead of installing everything individually. Basically the malware analysis version of Kali.

## **FlareVM: Arsenal of Tools**

The Windows equivalent of REMnux, since a lot of malware is obviously built to run on Windows so you sometimes need a Windows environment to actually analyse it properly (in an isolated VM obviously). Comes packed with tools for reverse engineering, forensics, and analysis.

## **Stuff worth remembering**

- REMnux and FlareVM basically cover both OS sides of malware analysis, good to know both exist depending on what you're looking at
- Always analyse malware in an isolated VM, snapshot before, never on a real machine
- CyberChef is one of those tools that just keeps being useful no matter what project I'm working on

---

That's the whole Cyber Security 101 path done. Onto AI Security next.
