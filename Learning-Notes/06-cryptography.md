# **Module 6: Cryptography** 

_TryHackMe notes - encryption, hashing, and how HTTPS actually works_ 

Last module in this set. Covers how data gets scrambled/protected, and why that matters for basically everything online. 

## **Symmetric vs Asymmetric encryption** 

- Symmetric - same key used to encrypt AND decrypt. Fast, but you've got to somehow share that key securely first (the tricky bit). AES is the big one used everywhere. 

- Asymmetric - uses a key pair, a public key (share with anyone) and a private key (keep secret). Encrypt with public key, only the matching private key can decrypt. Slower than symmetric but solves the "how do I share a key securely" problem. RSA is the classic example. 

In practice, most secure systems (like HTTPS) actually use both: asymmetric to securely exchange a symmetric key, then symmetric for the actual bulk of the data because it's way faster. 

## **Hashing (not the same as encryption, kept mixing this up)** 

Hashing turns data into a fixed-length string, but it's one-way, you can't reverse a hash back into the original data (unlike encryption, which is meant to be reversed with a key). 

- Used for checking integrity - if a file's hash changes, the file's been altered 

- Used for storing passwords - sites should store a hash of your password, not the actual password 

- Common algorithms: MD5 (old, broken, don't use), SHA-1 (also broken), SHA-256 (current standard) 

## **Digital signatures** 

Combines hashing + asymmetric encryption to prove a message genuinely came from who it says it did, and hasn't been tampered with. Sender hashes the message, encrypts that hash with their private key, and anyone can verify it using the sender's public key. 

## **TLS/SSL (how HTTPS actually works)** 

When you connect to an HTTPS site, there's a "handshake" where the browser and server agree on encryption methods and exchange keys (using asymmetric crypto), then switch to symmetric encryption for the actual browsing session. The little padlock icon in the browser = this handshake succeeded and the connection is encrypted. 

## **Stuff worth remembering for later rooms** 

- Weak/outdated hashing (MD5, SHA-1) shows up a lot in CTF rooms as a vulnerability to exploit 

- Encoding (like Base64) is NOT encryption, it's just a different way of representing data, anyone can decode it instantly, no key needed. Easy trap to fall into thinking Base64 = secure. 

