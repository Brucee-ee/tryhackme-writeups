# **Module 5: Networking** 

_TryHackMe notes - OSI model, TCP/IP, ports and protocols_ 

This is the module where it finally started clicking for me why the internet actually works. Two big models to know: OSI and TCP/IP. 

## **The OSI Model (7 layers)** 

Classic thing everyone has to memorise. A way of breaking down how data travels across a network into 7 layers: 

- 7. Application - the actual app/protocol the user interacts with (HTTP, FTP, DNS) 

- 6. Presentation - formatting/encryption of data (e.g. SSL/TLS happens around here) 

- 5. Session - manages the connection/session between devices 

- 4. Transport - reliable delivery, this is where TCP and UDP live 

- 3. Network - routing, this is where IP addresses and routers operate 

- 2. Data Link - how devices on the same local network talk (MAC addresses, switches) 

- 1. Physical - the actual cables/wifi/hardware 

Mnemonic the room actually gives: "Please Do Not Throw Spinach Pizza Away" (bottom to top, 1-7). I originally misremembered this as "sausage pizza" so double check I've actually got it right now lol. 

## **TCP/IP model (the simplified real-world version)** 

OSI is more theoretical, TCP/IP is what's actually used in practice, condensed into 4 layers: Application, Transport, Internet, Network Access. Same ideas, just squashed together. 

## **TCP vs UDP** 

- TCP - reliable, does a handshake first (SYN, SYN-ACK, ACK), makes sure packets arrive and in order. Used for stuff like web browsing where accuracy matters. 

- UDP - fire and forget, no handshake, faster but not guaranteed delivery. Used for stuff like video calls/gaming where speed matters more than perfect accuracy. 

## **Common protocols + ports (had to just memorise these)** 

- HTTP - port 80, plaintext web traffic 

- HTTPS - port 443, encrypted web traffic (uses TLS) 

- FTP - port 21, file transfer, plaintext (insecure) 

- SSH - port 22, secure remote login/command line access 

- DNS - port 53, translates domain names (google.com) into IP addresses 

- SMTP - port 25, sending emails 

**IP addresses and subnetting (the bit I need to redo a few times to actually stick)** An IP address like 192.168.1.10 identifies a device on a network. Subnet masks (like /24) define how big that network is, i.e. how many devices can fit in it. Still need more practice with subnetting maths tbh, worth revisiting this before Security+. 

