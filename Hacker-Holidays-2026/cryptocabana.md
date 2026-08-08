# CryptoCabana (TryHackMe, Hacker Holidays 2026 - Day 9) — Writeup

Doing Hacker Holidays this year and CryptoCabana was Day 9. Genuinely one of my favourites so far — no exploit code, no fuzzing, just proper "read everything, trust nothing" enumeration. Took me a while and I definitely went down a dead end partway through, but that's kind of the fun of it. Writing this up mostly for future-me but also because I didn't find loads of beginner-friendly writeups when I was stuck.

Quick context if you're new to this kind of room: it's an Azure cloud security challenge. No terminal on a box to break into — it's all about following a trail of Azure misconfigs (storage tokens, leaked creds, Key Vault permissions) until you land on a flag.

## The premise

Site's called CryptoCabana. Pretends to be a "secure" crypto seed phrase backup service — text box, paste your recovery phrase, hit "Back it up." Obviously not doing that with a real phrase, but the point of the room is to see what the site is doing behind that button before you even touch it.

## Step 1 — check the page source before touching anything

First instinct on literally any web-based room: don't interact with the form, look at what's shipped to the browser first. Opened DevTools → Sources → `app.js`.

![Viewing the page source in DevTools, showing the storage account name and SAS token hardcoded in app.js](images/01-source.png)

Straight away:

```js
const STORAGE_ACCOUNT = "cryptocabanaf5scjagc";
const BACKUPS_CONTAINER = "backups";
const BACKUP_SAS = "?sv=2022-11-02&ss=b&srt=sco&sp=rl&se=2099-12-31T23:59:59Z&st=2024-01-01T00:00:00Z&spr=https&sig=...";
```

So the "secure backup" is just client-side JS doing a raw PUT to Azure Blob Storage, using a SAS token that's sitting in plaintext for anyone to copy out of dev tools. I didn't know what half of a SAS token's query params meant going in, so I looked them up rather than skipping past them — that mattered later:

- `ss=b` — blob service only
- `srt=sco` — service + container + object scope (way more than "upload one file" needs)
- `sp=rl` — read + list permissions (again, more than an upload button should have)
- `se=2099-12-31` — expiry basically doesn't exist

That `srt=sco` + `sp=rl` combo means this token can browse and read stuff way outside the one container it's meant for. First misconfig, and the room's whole "in" point.

## Step 2 — see what else is in the storage account

Used the token to list every container on the account, not just `backups`:

```bash
az storage container list \
  --account-name cryptocabanaf5scjagc \
  --sas-token "sv=2022-11-02&ss=b&srt=sco&sp=rl&se=2099-12-31T23:59:59Z&st=2024-01-01T00:00:00Z&spr=https&sig=..." \
  --output table
```

![Terminal output listing three containers: $web, backups, and vault](images/02-containers.png)

```
$web
backups
vault
```

`$web` is just the static site hosting itself, `backups` is the one the app actually uses. `vault` isn't referenced anywhere in the front-end code — nothing on the page links to it, nothing in the JS calls it. That's the one worth poking at.

## Step 3 — see what's inside vault

```bash
az storage blob list --account-name cryptocabanaf5scjagc --container-name vault --sas-token "..." --output table
```

![Terminal output listing two blobs in the vault container: backup-service-account.json and seed_phrase.txt](images/03-vault-blobs.png)

```
backup-service-account.json
seed_phrase.txt
```

`seed_phrase.txt` felt like an obvious plant to waste time on (I didn't even bother opening it), `backup-service-account.json` sounded a lot more interesting. Downloaded and read it:

![Downloading backup-service-account.json and starting to cat it, showing the blob metadata](images/04-json-download.png)

![The rest of the JSON content, showing client_id, client_secret, key_vault_uri and tenant_id fields](images/05-json-full.png)

*(Heads up if you're reading this on GitHub: the client_secret in the text below is redacted since push protection flagged it, but it's still visible in plaintext in the screenshot above — GitHub's secret scanner only reads text, not image pixels, so screenshots don't get caught automatically. Worth remembering for any future writeups.)*

```json
{
  "client_id": "dbcf2923-e4eb-4b72-a0a4-688aa1185cf5",
  "client_secret": "<redacted>",
  "key_vault_name": "ccabana-kv-f5scjagc",
  "key_vault_uri": "https://ccabana-kv-f5scjagc.vault.azure.net/",
  "note": "CryptoCabana backup automation account. Rotate this if it ever leaves the vault. -- IT",
  "tenant_id": "8f8c5f8e-42d3-4ceb-97ad-241bbf446d6c"
}
```

Full Azure service principal credentials — client ID, client secret, tenant ID — sitting in blob storage that a public, overscoped SAS token can reach. The `note` field is basically IT going "please rotate this if it ever leaks" which, given I'm reading it right now, clearly didn't happen properly. Small thing but I liked that detail, it's a nice bit of storytelling for what's otherwise a dry misconfig.

Minor annoyance here: the tenant_id got cut off in my terminal window the first time I catted the file (screen wasn't wide enough), so I had to re-extract it with:

```bash
cat backup-service-account.json | python3 -c "import json,sys; print(json.load(sys.stdin)['tenant_id'])"
```

Small thing but worth remembering — always double check you've got the FULL value of a GUID/key before using it, wrapped terminal output will bite you.

## Step 4 — log in as the service principal

```bash
az login --service-principal \
  --username dbcf2923-e4eb-4b72-a0a4-688aa1185cf5 \
  --password '<redacted>' \
  --tenant 8f8c5f8e-42d3-4ceb-97ad-241bbf446d6c
```

![Extracting the full tenant_id via python, then the az login command and its successful JSON output showing the Az-Subs-CTF subscription](images/06-login.png)

Worked first try, logged in as `Az-Subs-CTF`. Small win, but a good checkpoint — if this had failed I'd have known something about the creds was wrong rather than the whole path being wrong.

## Step 5 — what's actually in the Key Vault

```bash
az keyvault secret list --vault-name ccabana-kv-f5scjagc --output table
```

![Terminal output listing four secrets: key-shard-1, key-shard-2, key-shard-3, and master-key with an expiry of 2020-01-01](images/07-secret-list.png)

Four secrets:

```
key-shard-1
key-shard-2
key-shard-3
master-key   (Expires: 2020-01-01)
```

`master-key` expiring in 2020 immediately jumped out — felt way too obvious to be a coincidence. Went for the three shards first since they seemed like the simpler grab:

![Terminal output showing the three shard values, with key-shard-2 returning a hint message instead of a value](images/08-shards.png)

```
key-shard-1 → THM{n0t_ur
key-shard-2 → "Rotated this after IT flagged it -- old value should still be recoverable if you know where to look."
key-shard-3 → ur_c01ns!}
```

So two clean flag chunks, and shard-2 gives you a riddle instead of a value. Half the flag plus a puzzle. Fun, but this is where I actually got stuck for a bit.

## Step 6 — the dead end (spent a good while here)

My first read of that shard-2 message was "old value should still be recoverable" = go check `master-key`, since its expiry date had already caught my eye. Made total sense at the time. Pulled its version history:

```bash
az keyvault secret list-versions --vault-name ccabana-kv-f5scjagc --name master-key \
  --query "[].{Version:id, Enabled:attributes.enabled, Expires:attributes.expires, Created:attributes.created}" \
  --output table
```

Only one version existed. Tried reading it by explicit version anyway, then tried backup, then tried just reading its attributes/tags — every single one came back Forbidden:

![Terminal output showing the master-key version list, followed by three separate Forbidden/ForbiddenByRbac errors for getSecret, backup, and attribute reads](images/09-masterkey-forbidden.png)

```
(Forbidden) Caller is not authorized to perform action on resource.
Inner error: { "code": "ForbiddenByRbac" }
```

Spent a bit too long here trying different actions against `master-key` assuming I just hadn't found the right angle yet. In hindsight the fact that literally every action was denied (not just "get" but backup and metadata too) was the tell that this secret wasn't reachable by design — it wasn't a puzzle to solve, it was a decoy. Also tried listing my own role assignments to see what I actually had access to, which came back completely empty — turns out the service principal doesn't have permission to read its own RBAC assignments either, so that was a dead end too.

## Step 7 — re-read the actual hint properly

Went back to shard-2's message and reread it slower: *"Rotated **this** after IT flagged it."* "This" is shard-2 talking about itself, not pointing at master-key. Felt a bit silly once I saw it, but that's the kind of thing that's obvious in hindsight and not at all obvious at 11pm three hours into a room.

Checked key-shard-2's own version history instead:

```bash
az keyvault secret list-versions --vault-name ccabana-kv-f5scjagc --name key-shard-2 \
  --query "[].{Version:id, Enabled:attributes.enabled, Expires:attributes.expires, Created:attributes.created}" \
  --output table
```

Two versions this time, a couple seconds apart:

```
3d6492d2c6f74123bc754a9ded22b2a0   created 01:05:05   (original)
c922c422ffb34671a902389c372314f1   created 01:05:07   (rotated, the hint text)
```

Grabbed the older one:

```bash
az keyvault secret show --vault-name ccabana-kv-f5scjagc --name key-shard-2 \
  --version 3d6492d2c6f74123bc754a9ded22b2a0 --query value -o tsv
```

![Terminal output showing the older version of key-shard-2 returning the value _k3ys_n0t_](images/10-shard2-oldversion.png)

```
_k3ys_n0t_
```

That's the missing chunk.

## Flag

All three shards in order:

```
THM{n0t_ur_k3ys_n0t_ur_c01ns!}
```

Genuinely happy with this one — "not your keys, not your coins" is a real crypto phrase, so having it be the literal flag text is a nice payoff for a room about a fake seed-phrase backup service.

## What I actually learned (not just "what the room teaches")

- Never assume "read the front-end code" is a boring first step — it's basically always where these rooms start, and I nearly rushed past it.
- Look up what SAS/token parameters actually mean rather than skimming past them. Understanding `srt=sco` + `sp=rl` up front is what told me to go enumerate containers instead of just using the token the "intended" way.
- Wrapped/truncated terminal output can genuinely cost you time — got caught out by a cut-off tenant_id and had to re-extract it properly.
- If every single action on a resource gets denied (not just the one you tried first), that's a decoy or a dead end, not a permissions puzzle to keep poking at — I lost a chunk of time on `master-key` before accepting that.
- Read hint text literally and slowly. "Rotated this" being self-referential seems obvious now but I completely missed it on the first pass.
- Rotating a Key Vault secret doesn't delete the old version — that's the whole crux of the room, and worth remembering for real Azure environments too, not just CTFs.
