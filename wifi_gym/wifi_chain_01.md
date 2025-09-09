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

## Anki Deck:
```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

A1q9dX7uY4	Basic (type in the answer)	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	Which Linux command lists all available network interfaces and shows their wireless-specific properties such as ESSID, mode, frequency, and Tx power?	<code>iwconfig</code>	wifi
B7t2mJ6fQ8	Basic	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	In the <code>iwconfig</code> output, what name is usually assigned to the Wi-Fi interface?	<code>wlan0</code> (though sometimes <code>wlp2s0</code>, <code>wlan1</code>, etc.)	wifi
C5r1vN8kM2	Basic	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	What do <code>eth0</code> and <code>lo</code> represent in <code>iwconfig</code> output?	<code>eth0</code> = Ethernet interface, <code>lo</code> = loopback interface	wifi
D3y4bK2pL9	Basic (type in the answer)	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	What command shows the physical device (PHY), interface name, driver, and chipset of detected wireless cards?	<code>sudo airmon-ng</code>	wifi
E6u9zH5sW1	Basic	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	Why is knowing the chipset of a Wi-Fi card important for penetration testing?	Because only some chipsets support monitor mode and packet injection, which are essential for Wi-Fi attacks.	wifi
F8p3cQ7nR0	Basic (type in the answer)	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	Which command displays the currently configured regulatory domain of a Wi-Fi card?	<code>iw reg get</code>	wifi
G2o6mD1kT5	Basic	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	What is the default regulatory domain shown as in <code>iw reg get</code>?	<code>country 00: DFS-UNSET</code>	wifi
H9n7xJ4rV2	Basic (type in the answer)	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	Which command sets the Wi-Fi card’s regulatory domain to the United States?	<code>sudo iw reg set US</code>	wifi
I4e0fC8aY6	Basic	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	Why is it important to set the correct regulatory domain on a Wi-Fi interface?	Because it determines allowed transmit power and frequency ranges; incorrect settings may reduce performance or break local laws.	wifi
J1d2qK9wZ8	Basic	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	What is the typical default Tx power value for most Wi-Fi cards?	<code>20 dBm</code>	wifi
K5m8hF2rX7	Cloze (type in the answer)	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	Which sequence of commands increases the Wi-Fi card’s Tx power to 30 dBm?<br><br>{{c1::sudo ifconfig wlan0 down}}<br>{{c1::sudo iwconfig wlan0 txpower 30}}<br>{{c1::sudo ifconfig wlan0 up}}		wifi
L0p4uN7tE3	Basic	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	What risks exist when increasing Tx power beyond the regulatory limit?	It may be illegal, damage the card due to overheating, or simply fail if the driver blocks it.	wifi
M3s1vB6yQ4	Basic (type in the answer)	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	What command shows all capabilities of a Wi-Fi driver and chipset, including supported modes (e.g., monitor) and ciphers?	<code>iw list</code>	wifi
N2w9kD5aL6	Basic	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	Which Wi-Fi interface mode allows capturing all packets in the air, even those not addressed to the host?	Monitor mode	wifi
O7e8qJ3hR1	Basic	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	Which Wi-Fi interface mode is required for setting up a rogue AP or Evil Twin attack?	Master mode (AP mode)	wifi
P6t0xK4mV9	Basic	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	Which Wi-Fi interface mode is typically used for mesh networking attacks or experiments?	Mesh mode	wifi
Q5r3zL8cJ7	Basic (type in the answer)	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	What command scans for nearby Wi-Fi networks and filters output to show cell, quality, ESSID, and IEEE information?	<code>iwlist wlan0 scan | grep 'Cell\|Quality\|ESSID\|IEEE'</code>	wifi
R1m6yD9bQ2	Basic	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	What does the <code>Quality</code> field represent in <code>iwlist</code> scan results?	The signal quality of the detected network.	wifi
S4e2jF0kM8	Basic	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	What does the <code>ESSID</code> field represent in Wi-Fi scanning outputs?	The human-readable Wi-Fi network name.	wifi
T8y7nB2dP5	Basic (type in the answer)	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	Which command lists all available channels supported by a given wireless card?	<code>iwlist wlan0 channel</code>	wifi
U6o4vN3hR9	Cloze (type in the answer)	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	Which commands lock the Wi-Fi interface to channel 11?<br><br>{{c1::sudo ifconfig wlan0 down}}<br>{{c1::sudo iwconfig wlan0 channel 11}}<br>{{c1::sudo ifconfig wlan0 up}}		wifi
V9p1tJ5mQ0	Cloze (type in the answer)	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	Which commands set the Wi-Fi card to the frequency 5.52 GHz?<br><br>{{c1::sudo ifconfig wlan0 down}}<br>{{c1::sudo iwconfig wlan0 freq 5.52G}}<br>{{c1::sudo ifconfig wlan0 up}}		wifi
W2k8dH6rL3	Basic	Ops::WiFi Gym::Chain 01: Interface Setup & Configuration	Why would a penetration tester lock their interface to a single channel instead of letting it hop across all channels?	To reliably capture handshakes or traffic from a specific target AP without missing packets during channel hopping.	wifi
rA7kX2pQmN Basic (type in the answer) Ops::WiFi Gym::Chain 01: Interface Setup & Configuration Which command locks the monitor interface <code>wlan0mon</code> to channel 1 using nl80211? <code>sudo iw dev wlan0mon set channel 1</code> wifi
```