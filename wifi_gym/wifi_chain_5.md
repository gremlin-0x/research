# Chain #5: Decrypting Captures

This chain focuses on analyzing encrypted packet captures by decrypting them with **Airdecap-ng**. This step is essential once you have already obtained a key (WEP/WPA/WPA2). Without decryption, the capture will only show 802.11 management/data frames and MAC addresses — not the underlying protocols (TCP, HTTP, etc.).

---

## Step 1: Identify What You Have Captured

Before decryption, confirm what kind of network data you have:

- **Open network capture**: Already unencrypted but includes unnecessary wireless headers.
- **WEP-encrypted capture**: Requires a WEP key (hexadecimal).
- **WPA/WPA2-encrypted capture**: Requires the passphrase (PSK) or Pairwise Master Key.

Check with `aircrack-ng` or inspect the `.cap` file in **Wireshark** to see what you’re working with.

---

## Step 2: Run Airdecap-ng

**General syntax:**
```bash
airdecap-ng [options] <capture-file>
```

Key options:
- `-l` → Do not remove 802.11 headers (keep them in the output file).
- `-b <BSSID>` → Focus on traffic from a specific access point.
- `-w <WEP-key>` → Decrypt using WEP key (hex).
- `-p <passphrase>` → Decrypt using WPA/WPA2 passphrase.
- `-e <ESSID>` → Specify the target ESSID (used with WPA/WPA2).
- `-k <PMK>` → Use Pairwise Master Key directly (hex).

The tool produces a new file with the suffix **`-dec.cap`**.

---

## Step 3: Remove Wireless Headers from Open Networks

When analyzing open (unencrypted) captures, you may still want to strip out unnecessary wireless headers:

```bash
sudo airdecap-ng -b 00:14:6C:7A:41:81 opencapture.cap
```

- `-b 00:14:6C:7A:41:81` → Limits processing to the target AP.
- `opencapture.cap` → The raw capture file.

Result → `opencapture-dec.cap` without unnecessary 802.11 headers.

---

## Step 4: Decrypt WEP-encrypted Captures

If you cracked a WEP key with `aircrack-ng`, use it here:

```bash
sudo airdecap-ng -w 1234567890ABCDEF HTB-01.cap
```

- `-w 1234567890ABCDEF` → WEP key (hexadecimal only).
- `HTB-01.cap` → Encrypted capture.

Result → `HTB-01-dec.cap` with decrypted traffic.

---

## Step 5: Decrypt WPA/WPA2-encrypted Captures

If you have the passphrase, you must provide the SSID as well:

```bash
sudo airdecap-ng -p 'Password123' -e "HackTheBox" HTB-01.cap
```

- `-p 'Password123'` → WPA/WPA2 passphrase.
- `-e "HackTheBox"` → ESSID name (must match exactly).
- `HTB-01.cap` → Encrypted capture.

Result → `HTB-01-dec.cap` with decrypted packets.

---

## Step 6: Verify Decryption in Wireshark

- **Before decryption**: Only see `802.11` frames, MAC addresses, and management frames (Beacon, Probe, etc.).
- **After decryption**: See actual protocols (TCP, UDP, HTTP, DHCP, ARP, etc.) with readable source/destination IPs.

---

## Anki Deck:

```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

A1b2C3d4E5	Basic	Ops::WiFi Gym::Chain 5: Decrypting Captures	What is the purpose of Airdecap-ng in Wi-Fi penetration testing?	To decrypt WEP/WPA/WPA2 encrypted capture files and/or strip unnecessary wireless headers from open network captures.	wifi
F6g7H8i9J0	Basic	Ops::WiFi Gym::Chain 5: Decrypting Captures	What suffix does Airdecap-ng append to output files after processing?	<code>-dec.cap</code>	wifi
K1l2M3n4O5	Basic	Ops::WiFi Gym::Chain 5: Decrypting Captures	Which Aircrack-ng tool is specifically used to decrypt capture files after keys are obtained?	Airdecap-ng	wifi

P6q7R8s9T0	Basic (type in the answer)	Ops::WiFi Gym::Chain 5: Decrypting Captures	In Airdecap-ng, which option prevents removal of 802.11 headers?	<code>-l</code>	wifi
U1v2W3x4Y5	Basic (type in the answer)	Ops::WiFi Gym::Chain 5: Decrypting Captures	In Airdecap-ng, which option restricts output to a specific AP MAC address?	<code>-b &lt;BSSID&gt;</code>	wifi
Z6a7B8c9D0	Basic (type in the answer)	Ops::WiFi Gym::Chain 5: Decrypting Captures	In Airdecap-ng, which option provides a WEP key for decryption?	<code>-w &lt;WEP-key&gt;</code>	wifi
E1f2G3h4I5	Basic (type in the answer)	Ops::WiFi Gym::Chain 5: Decrypting Captures	In Airdecap-ng, which option provides a WPA/WPA2 passphrase?	<code>-p &lt;passphrase&gt;</code>	wifi
J6k7L8m9N0	Basic (type in the answer)	Ops::WiFi Gym::Chain 5: Decrypting Captures	In Airdecap-ng, which option specifies the ESSID of the target network (used with WPA/WPA2)?	<code>-e &lt;ESSID&gt;</code>	wifi
Q1r2S3t4U5	Basic (type in the answer)	Ops::WiFi Gym::Chain 5: Decrypting Captures	In Airdecap-ng, which option provides the Pairwise Master Key directly?	<code>-k &lt;PMK&gt;</code>	wifi

V6w7X8y9Z0	Basic (type in the answer)	Ops::WiFi Gym::Chain 5: Decrypting Captures	Which Airdecap-ng command removes wireless headers from an unencrypted capture focused on AP <code>00:14:6C:7A:41:81</code> in file <code>opencapture.cap</code>?	<code>sudo airdecap-ng -b 00:14:6C:7A:41:81 opencapture.cap</code>	wifi
B1c2D3e4F5	Basic (type in the answer)	Ops::WiFi Gym::Chain 5: Decrypting Captures	Which Airdecap-ng command decrypts <code>HTB-01.cap</code> using WEP key <code>1234567890ABCDEF</code>?	<code>sudo airdecap-ng -w 1234567890ABCDEF HTB-01.cap</code>	wifi
G6h7I8j9K0	Basic (type in the answer)	Ops::WiFi Gym::Chain 5: Decrypting Captures	Which Airdecap-ng command decrypts <code>HTB-01.cap</code> using WPA passphrase <code>Password123</code> and SSID <code>HackTheBox</code>?	<code>sudo airdecap-ng -p 'Password123' -e "HackTheBox" HTB-01.cap</code>	wifi

L1m2N3o4P5	Basic	Ops::WiFi Gym::Chain 5: Decrypting Captures	Why must both passphrase and ESSID be provided when decrypting WPA/WPA2 captures with Airdecap-ng?	Because the passphrase and ESSID are combined to derive the Pairwise Master Key (PMK).	wifi
S6t7U8v9W0	Basic	Ops::WiFi Gym::Chain 5: Decrypting Captures	What is the difference between decrypting WEP and WPA/WPA2 with Airdecap-ng?	WEP requires the raw hexadecimal key; WPA/WPA2 requires the passphrase and ESSID to derive the key.	wifi
```