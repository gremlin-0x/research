# Chain #5: Decrypting Captures

This chain focuses on analyzing encrypted packet captures by decrypting them with **Airdecap-ng**. This step is essential once you have already obtained a key (WEP/WPA/WPA2). Without decryption, the capture will only show 802.11 management/data frames and MAC addresses — not the underlying protocols (TCP, HTTP, etc.).

---

## Step 1: Identify What You Have Captured

Before decryption, confirm what kind of network data you have:

- **Open network capture**: Already unencrypted but includes unnecessary wireless headers.
- **WEP-encrypted capture**: Requires a WEP key (hexadecimal).
- **WPA/WPA2-encrypted capture**: Requires the passphrase (PSK) or Pairwise Master Key.

Check with `aircrack-ng` or inspect the `.cap` file in **Wireshark** to see what you’re working with.

---

## Step 2: Run Airdecap-ng

**General syntax:**
```bash
airdecap-ng [options] <capture-file>
```

Key options:
- `-l` → Do not remove 802.11 headers (keep them in the output file).
- `-b <BSSID>` → Focus on traffic from a specific access point.
- `-w <WEP-key>` → Decrypt using WEP key (hex).
- `-p <passphrase>` → Decrypt using WPA/WPA2 passphrase.
- `-e <ESSID>` → Specify the target ESSID (used with WPA/WPA2).
- `-k <PMK>` → Use Pairwise Master Key directly (hex).

The tool produces a new file with the suffix **`-dec.cap`**.

---

## Step 3: Remove Wireless Headers from Open Networks

When analyzing open (unencrypted) captures, you may still want to strip out unnecessary wireless headers:

```bash
sudo airdecap-ng -b 00:14:6C:7A:41:81 opencapture.cap
```

- `-b 00:14:6C:7A:41:81` → Limits processing to the target AP.
- `opencapture.cap` → The raw capture file.

Result → `opencapture-dec.cap` without unnecessary 802.11 headers.

---

## Step 4: Decrypt WEP-encrypted Captures

If you cracked a WEP key with `aircrack-ng`, use it here:

```bash
sudo airdecap-ng -w 1234567890ABCDEF HTB-01.cap
```

- `-w 1234567890ABCDEF` → WEP key (hexadecimal only).
- `HTB-01.cap` → Encrypted capture.

Result → `HTB-01-dec.cap` with decrypted traffic.

---

## Step 5: Decrypt WPA/WPA2-encrypted Captures

If you have the passphrase, you must provide the SSID as well:

```bash
sudo airdecap-ng -p 'Password123' -e "HackTheBox" HTB-01.cap
```

- `-p 'Password123'` → WPA/WPA2 passphrase.
- `-e "HackTheBox"` → ESSID name (must match exactly).
- `HTB-01.cap` → Encrypted capture.

Result → `HTB-01-dec.cap` with decrypted packets.

---

## Step 6: Verify Decryption in Wireshark

- **Before decryption**: Only see `802.11` frames, MAC addresses, and management frames (Beacon, Probe, etc.).
- **After decryption**: See actual protocols (TCP, UDP, HTTP, DHCP, ARP, etc.) with readable source/destination IPs.