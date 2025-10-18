# Chain #2: Switching Interface Modes

Wireless interfaces can operate in multiple modes, each serving a different purpose in Wi-Fi communication. For pentesting, knowing how to switch between them and when to use each is crucial.

---
## Step 1: Managed Mode (Default Client Mode)
- **Purpose**: Connects your interface as a normal client (station) to access points.
- **Use Case**: Returning to a normal state after monitor mode or connecting to a target network legitimately.

**Commands:**
```bash
sudo ifconfig wlan0 down
sudo iwconfig wlan0 mode managed
sudo iwconfig wlan0 essid <NetworkName>
sudo ifconfig wlan0 up
```
- Bring interface down → change mode → set ESSID → bring interface up.
- Verify mode with:
```bash
iwconfig
```

---

## Step 2: Ad-Hoc Mode (Peer-to-Peer)
- **Purpose**: Direct communication between wireless devices without an AP.
- **Use Case**: Testing peer-to-peer links or mesh-like systems.

**Commands:**
```bash
sudo iwconfig wlan0 mode ad-hoc
sudo iwconfig wlan0 essid <NetworkName>
```
- No central AP — devices directly connect using the same ESSID.

---

## Step 3: Master Mode (Access Point Mode)
- **Purpose**: Turns your interface into an AP.
- **Use Case**: Rogue AP/Evil Twin attacks.
- **Requirement**: Needs a management daemon (like `hostapd`).

**Minimal `hostapd` config (open network):**
```
interface=wlan0
driver=nl80211
ssid=HTB-Hello-World
channel=2
hw_mode=g
```

**Start AP:**
```bash
sudo hostapd open.conf
```

---

## Step 4: Mesh Mode
- **Purpose**: Joins/runs a self-configuring mesh network.
- **Use Case**: Testing enterprise/large deployments.

**Command:**
```bash
sudo iw dev wlan0 set type mesh
```
- Check with `iwconfig`. If supported, the interface will accept mesh configuration.

---

## Step 5: Monitor Mode
- **Purpose**: Captures **all packets** in range, regardless of recipient.
- **Use Case**: Essential for pentesting (packet capture, handshake capture, traffic analysis).

**Manual method:**
```bash
sudo ifconfig wlan0 down
sudo iw wlan0 set monitor control
sudo ifconfig wlan0 up
```

**Verify:**
```bash
iwconfig
```
- Should show: `Mode:Monitor`

---

## Pentester’s Checklist for Modes
- **Need to connect to a network?** → Use **Managed**.
- **Need peer-to-peer (without AP)?** → Use **Ad-Hoc**.
- **Need to host a malicious AP?** → Use **Master**.
- **Testing mesh/backhaul setups?** → Use **Mesh**.
- **Capturing raw traffic?** → Use **Monitor**.