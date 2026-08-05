# Pre Security — Notes

Working through TryHackMe's Pre Security path. Covers the actual fundamentals before getting into anything offensive, so worth having solid rather than skipping to the fun stuff.

## Introduction to Cyber Security

Split roughly into two camps: offensive security (red team, pentesting, finding weaknesses before attackers do) and defensive security (blue team, detection, incident response, keeping things running). Most orgs need both, and a lot of roles blend the two (purple team). Careers in cyber span way beyond "hacker" as a job title, things like SOC analyst, threat intel, GRC, forensics, security engineering all count.

## Computer Fundamentals

- **Inside a computer**: CPU does the actual processing, RAM is fast short-term memory that clears on shutdown, storage (HDD/SSD) is slow but persistent. Motherboard ties it all together.
- **Computer types**: servers vs desktops vs embedded systems, all different tradeoffs of power, portability, purpose.
- **Client-server model**: clients request, servers respond. Basically every web interaction is this pattern, your browser is the client, the site's backend is the server.
- **Virtualisation**: running an OS inside another OS via a hypervisor (VMware, VirtualBox, Hyper-V). Lets you isolate environments, which is exactly why every CTF box runs as a VM rather than on real hardware.
- **Cloud computing**: renting compute/storage instead of owning hardware. IaaS/PaaS/SaaS distinction matters, IaaS gives you raw infrastructure (VMs, storage), PaaS gives you a platform to deploy on without managing servers, SaaS is just using someone else's finished software.

## Operating Systems Basics

- OS sits between hardware and applications, manages processes, memory, files, permissions.
- **Windows** dominates enterprise desktop, has its own permission model, registry, and a completely different CLI ecosystem to Linux.
- **Linux CLI basics**: navigating with `cd`, `ls`, `pwd`, file ops with `cp`, `mv`, `rm`, permissions with `chmod`/`chown`. This stuff becomes second nature fast once you're doing rooms regularly, basically muscle memory now from Beach Bar and the others.
- **Windows CLI basics**: cmd and PowerShell, different syntax to Linux but same underlying idea, navigating, running programs, managing files.
- **OS security**: patching, least privilege, disabling unnecessary services. Most real breaches trace back to something boring like an unpatched system or an overprivileged account, not some elite zero-day.

## Software Basics

- **Data representation**: everything's binary underneath, bits and bytes, hex is just a more human-friendly way of writing binary in groups of 4 bits.
- **Data encoding**: encoding (base64, URL encoding) isn't encryption, it's just a different representation, fully reversible with no key needed. Important distinction that trips people up early on, encoding hides nothing from someone who knows to decode it.
- **Python/JavaScript simple demos**: enough to read and lightly modify scripts, which matters a lot for CTFs since so many rooms involve reading someone else's exploit code or tweaking a payload (exactly what we did with the YAML payloads on Beach Bar).
- **Database SQL basics**: SELECT/INSERT/UPDATE/DELETE, basic structure of tables and queries. Foundation for understanding SQL injection later even though this module itself is just the legitimate use of SQL.

## Network Fundamentals

- **What is networking**: devices talking to each other, needs a shared protocol to make sense of the exchange.
- **LAN**: local network, same physical/logical space, e.g. your house or office network.
- **OSI model**: 7 layers (Physical, Data Link, Network, Transport, Session, Presentation, Application), a conceptual model for how data moves from an application on one machine to an application on another. Rarely used in that exact form day to day, but it's the shared vocabulary everyone in networking uses to describe where a problem sits, "that's a layer 2 issue" etc.
- **Packets and frames**: data gets broken into packets at the network layer, wrapped in frames at the data link layer, each with headers containing addressing info (IP addresses for packets, MAC addresses for frames).
- **Extending your network**: switches operate at layer 2, routers at layer 3, this is how networks connect to other networks (and eventually the internet).

## How The Web Works

- **DNS**: translates domain names to IP addresses, hierarchical system (root servers, TLD servers, authoritative servers). Every time you type a URL this resolution happens before anything else can load.
- **HTTP**: the protocol web traffic runs on, methods (GET, POST, PUT, DELETE), status codes (200 ok, 404 not found, 500 server error), headers carrying metadata. HTTPS is the same thing wrapped in TLS encryption.
- **How websites work**: frontend (what you see, HTML/CSS/JS) vs backend (server-side logic, databases). This split is exactly why something like Beach Bar's YAML import bug lives on the backend, invisible from the frontend until you go looking.
- **Putting it together**: DNS resolves domain, browser sends HTTP request, server processes it (maybe hitting a database), sends back a response, browser renders it. Every web-based room I do is really just poking at some step in this chain.

## Attacks and Defenses

- **CIA Triad**: Confidentiality, Integrity, Availability. The three things security is fundamentally trying to protect, and most attacks map to breaking one of these (data leak = confidentiality, tampering = integrity, DoS = availability).
- **Cryptography concepts**: symmetric (same key encrypts/decrypts, fast, key distribution is the hard problem) vs asymmetric (public/private key pairs, slower but solves the distribution problem), hashing (one-way, for integrity checks and password storage, not encryption since it can't be reversed).
- **Become a Hacker / Become a Defender**: two intro modules framing the offensive vs defensive mindset, not much technical content, more scene-setting for the rest of the platform.

## Takeaways

Nothing here is exploit-specific, it's all just the vocabulary and mental models that make the actual hands-on rooms make sense. Worth having this solid before rushing into more boot2root stuff, since half of understanding something like the Beach Bar YAML bug is already knowing how HTTP and the client-server model work in the first place.
