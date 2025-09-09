# Chain #6: Cracking Keys

This chain covers how to use **Aircrack-ng** to crack WEP and WPA/WPA2 keys once you’ve already captured the necessary packets (IVs or handshakes). The process is fully offline: no live interaction with access points after capture.

---

## Step 1: Benchmarking Your System

Before cracking, test CPU performance to estimate cracking speed.

```bash
aircrack-ng -S
```
- `-S` → Benchmark mode, tests key-cracking speed.
- Output: keys/second rate (k/s). Higher is better.

**Why this matters**: Cracking WPA with a large wordlist can take hours/days. Knowing your system’s speed lets you plan accordingly.

---

## Step 2: Cracking WEP Keys

### 2.1 Capture IVs with Airodump-ng

WEP cracking depends on Initialization Vectors (IVs). You need thousands to succeed.

```bash
airodump-ng --ivs -w capture wlan0mon
```
- `--ivs` → Save only IVs (smaller file size, faster analysis).
- `-w capture` → Output file prefix.

### 2.2 Run Aircrack-ng with Korek Method

```bash
aircrack-ng -K capture.ivs
```
- `-K` → Use Korek WEP attack (collection of statistical cracking methods).
- `capture.ivs` → File containing IVs.

**Output**: Displays tested keys until WEP key is found.

**Scenario Note**:
- If you don’t have enough IVs, combine with Aireplay-ng ARP injection attacks to speed up IV generation.

---

## Step 3: Cracking WPA/WPA2 Keys

### 3.1 Capture a Handshake

Use Airodump-ng to capture the WPA 4-way handshake.
- Deauthentication attacks with Aireplay-ng help force clients to reconnect, generating the handshake.

```bash
airodump-ng -c <channel> --bssid <AP_MAC> -w capture wlan0mon
```
- `-c <channel>` → Stay on target channel.
- `--bssid <AP_MAC>` → Lock on specific AP.
- `-w capture` → Save packets.

Look for `WPA handshake: <BSSID>` in top-right corner of output.

### 3.1b Extract for Hashcat (mode 22000)

```
# Extract WPA handshake/PMKID into Hashcat's 22000 format
hcxpcapngtool -o hackme.22000 hackme-01.cap
```
- Produces a `.22000` file for `hashcat -m 22000`. Faster/cleaner than older formats.
### 3.2 Run Aircrack-ng with Wordlist

```bash
aircrack-ng capture.cap -w /path/to/wordlist.txt
```
- `capture.cap` → File with captured handshake.
- `-w /path/to/wordlist.txt` → Dictionary file with possible passwords.

**Scenario Notes**:
- WPA cracking requires a wordlist. Pure brute-force (all combinations) is impractical.
- If handshake is partial (missing packets), Aircrack-ng may still crack if it has packets 2+3 or 3+4.

### 3.2b Legacy: convert to HCCAPX for Hashcat (mode 2500)

```
# Legacy export for Hashcat mode 2500 (.hccapx)
aircrack-ng capture.cap -J hackme
# -> hackme.hccapx (use: hashcat -m 2500 hackme.hccapx /path/to/wordlist.txt)
```
- Useful when following older guides that still expect `.hccapx`.
### 3.3 PMKID-Based Cracking (Alternative)

Some WPA networks allow PMKID extraction without clients.

```bash
aircrack-ng capture.cap -w /path/to/wordlist.txt
```
- Works the same way if capture contains PMKID.

---

## Step 4: Verifying the Key

Once a key is found, Aircrack-ng will display:
- **KEY FOUND! [ password ]**
- Derived Master/Transient keys

You can then test it by connecting to the network (see Chain #7).

---

## Anki Deck:

```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6
n1A2b3C4d5	Basic	Ops::WiFi Gym::Chain 6: Cracking Keys	What is an Initialization Vector (IV) in WEP cracking?	A per-packet random value used with the WEP key. Collecting many IVs allows statistical attacks to recover the WEP key.	wifi
f6G7h8I9j0	Basic	Ops::WiFi Gym::Chain 6: Cracking Keys	What is a WPA 4-way handshake?	An authentication exchange between client and AP that confirms both know the pre-shared key and generates encryption keys.	wifi
k1L2m3N4o5	Basic	Ops::WiFi Gym::Chain 6: Cracking Keys	What is a PMKID in WPA/WPA2 networks?	A Pairwise Master Key Identifier that can be captured directly from the AP without a client, enabling offline cracking.	wifi
p6Q7r8S9t0	Basic (type in the answer)	Ops::WiFi Gym::Chain 6: Cracking Keys	What does the <code>-K</code> option do in Aircrack-ng?	Invokes the Korek statistical attacks for WEP key recovery.	wifi
u1V2w3X4y5	Basic	Ops::WiFi Gym::Chain 6: Cracking Keys	Why is WPA cracking wordlist-based instead of statistical like WEP?	WPA uses strong per-session key derivation (PBKDF2 with SHA1) that cannot be cracked statistically; only dictionary attacks are feasible.	wifi
z6A7b8C9d0	Basic (type in the answer)	Ops::WiFi Gym::Chain 6: Cracking Keys	What Aircrack-ng command benchmarks cracking speed?	<code>aircrack-ng -S</code>	wifi
e1F2g3H4i5	Basic (type in the answer)	Ops::WiFi Gym::Chain 6: Cracking Keys	What Airodump-ng command captures only WEP IVs?	<code>airodump-ng --ivs -w capture wlan0mon</code>	wifi
j6K7l8M9n0	Basic (type in the answer)	Ops::WiFi Gym::Chain 6: Cracking Keys	What Aircrack-ng command cracks WEP IVs with Korek attacks?	<code>aircrack-ng -K capture.ivs</code>	wifi
o1P2q3R4s5	Basic (type in the answer)	Ops::WiFi Gym::Chain 6: Cracking Keys	What Airodump-ng command captures a WPA handshake for a specific AP?	<code>airodump-ng -c &lt;channel&gt; --bssid &lt;AP_MAC&gt; -w capture wlan0mon</code>	wifi
t6U7v8W9x0	Basic (type in the answer)	Ops::WiFi Gym::Chain 6: Cracking Keys	What Aircrack-ng command cracks a WPA handshake with a wordlist?	<code>aircrack-ng capture.cap -w /path/to/wordlist.txt</code>	wifi
hN2cR7yWpK	Basic (type in the answer)	Ops::WiFi Gym::Chain 6: Cracking Keys	Which command extracts WPA/PMKID material into Hashcat's 22000 format from <code>hackme-01.cap</code>?	<code>hcxpcapngtool -o hackme.22000 hackme-01.cap</code>	wifi
zT5uL1aGxD	Basic (type in the answer)	Ops::WiFi Gym::Chain 6: Cracking Keys	Which Aircrack-ng option exports an <code>.hccapx</code> file for Hashcat legacy mode 2500?	<code>-J</code> (e.g., <code>aircrack-ng capture.cap -J hackme</code>)	wifi
```