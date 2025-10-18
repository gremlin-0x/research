# Chain #1: Interface Setup & Configuration

This chain ensures your Wi-Fi interface is fully prepared for penetration testing: capable of monitor mode, packet injection, correct Tx power, and tuned to the right region/frequency.

---

## Step 1: Identify available interfaces
```bash
iwconfig
```
- Lists all network interfaces and their wireless capabilities.
- Example outputs:
  - `wlan0` → your Wi-Fi interface.
  - `eth0` → Ethernet, ignore.
  - `lo` → loopback, ignore.

At this point, you just need to know the interface name (commonly `wlan0`, but could be `wlp2s0`, `wlan1`, etc.).

---

## Step 2: Verify driver and chipset
```bash
sudo airmon-ng
```
- Displays:
  - **PHY** → physical device number.
  - **Interface** → name (e.g., `wlan0`).
  - **Driver** → kernel driver in use (e.g., `rt2800usb`).
  - **Chipset** → actual chipset (e.g., `Ralink RT3070`).

Why this matters:
- Not all drivers support **monitor mode** and **packet injection**, which are mandatory for Wi-Fi attacks.
- Research your chipset (e.g., on Airgeddon’s Wi-Fi adapter list) before relying on it.

---

## Step 3: Check regulatory domain (region)
```bash
iw reg get
```
- Shows what region your card is currently configured for.
- Example:
  - `country 00: DFS-UNSET` → generic default, often limited to **20 dBm Tx power**.
  - `country US` → allows higher Tx power on certain bands.

**IF your card is limited to DFS-UNSET → THEN set your real region:**
```bash
sudo iw reg set US   # Replace US with your country code
iw reg get           # Verify the change
```

Note: Changing to a different country can be illegal depending on local laws.

---

## Step 4: Adjust Tx Power (Signal Strength)
Check current Tx power:
```bash
iwconfig
```
- Look for `Tx-Power=20 dBm` (default).

**IF you need stronger range (e.g., for distant APs):**
```bash
sudo ifconfig wlan0 down
sudo iwconfig wlan0 txpower 30
sudo ifconfig wlan0 up
iwconfig
```
- Now `Tx-Power=30 dBm`.

Warnings:
- Some cards won’t accept this (driver-limited).
- Some cards *can* transmit at higher power but lack cooling → risk of damage.
- Illegal in many jurisdictions to exceed regulated limits.

---

## Step 5: Check driver capabilities in depth
```bash
iw list
```
This command gives a **capability dump** of your card. Look for:
- **Supported interface modes** → e.g.:
  - `* managed` (normal Wi-Fi use).
  - `* monitor` (essential for pentesting).
  - `* AP` (required for Rogue AP / Evil Twin).
  - `* mesh` or `* ad-hoc` (used in mesh exploitation).
- **Supported bands** → 2.4 GHz and/or 5 GHz.
- **Supported ciphers** → whether WPA3 (SAE) is supported.

This prevents wasted effort: if your card doesn’t support WPA3 or monitor mode, no amount of command tweaking will fix it.

---

## Step 6: Scan available networks
Quick Linux-native scan:
```bash
iwlist wlan0 scan | grep 'Cell\|Quality\|ESSID\|IEEE'
```
- Outputs:
  - **Cell** = unique AP.
  - **Quality/Signal level** = signal strength.
  - **ESSID** = Wi-Fi network name.
  - **IEEE** = protocol version (e.g., WPA2).

Why use this:
- Simple check that your card is working *before* moving into more advanced tools (like airodump-ng).

---

## Step 7: Change channel or frequency
By default, your card scans all channels. Sometimes you need to **lock it to one channel** (e.g., handshake capture).

**List available channels:**
```bash
iwlist wlan0 channel
```

**Set specific channel:**
```bash
sudo ifconfig wlan0 down
sudo iwconfig wlan0 channel 11
sudo ifconfig wlan0 up
```

**Set frequency directly (instead of channel):**
```bash
sudo ifconfig wlan0 down
sudo iwconfig wlan0 freq 5.52G
sudo ifconfig wlan0 up
```

**Alternative (modern, nl80211-native) channel lock for a monitor iface**:
```
sudo iw dev wlan0mon set channel 1
```
- Use this when `airodump-ng` and `aireplay-ng` drift; it hard-locks the monitor interface to the target channel.

Use case:
- Sticking to channel 6 to capture a specific WPA handshake.
- Locking to 5.52 GHz (Channel 104) when working on 5 GHz APs.

---

## End State of Chain #1
By completing this chain, you now have:
- Identified your Wi-Fi interface and driver.
- Ensured your card supports monitor mode and injection.
- Tuned regulatory domain + Tx power for better performance.
- Verified your card’s real capabilities.
- Scanned visible networks.
- Locked your card onto a specific channel or frequency if needed.