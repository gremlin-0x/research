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

## Anki Deck:
```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

X1a7pD4mL9	Basic	Ops::WiFi Gym::Chain 2: Switching Interface Modes	In Wi-Fi networking, what is *Managed Mode*?	The default client mode where a wireless interface connects to an access point (AP) as a station.	wifi
Y6f3qB9kT2	Basic	Ops::WiFi Gym::Chain 2: Switching Interface Modes	What is *Ad-Hoc Mode* in Wi-Fi networking?	A peer-to-peer mode where wireless devices connect directly without an access point.	wifi
Z2r8mN5jH1	Basic	Ops::WiFi Gym::Chain 2: Switching Interface Modes	What is *Master Mode* in Wi-Fi networking?	The mode that turns an interface into an access point (AP), requiring a management daemon like hostapd.	wifi
W9u4cE7kR3	Basic	Ops::WiFi Gym::Chain 2: Switching Interface Modes	What is *Mesh Mode* in Wi-Fi networking?	A mode that allows the interface to join a self-configuring, routing mesh network for extended coverage.	wifi
V5o0dJ8nQ6	Basic	Ops::WiFi Gym::Chain 2: Switching Interface Modes	What is *Monitor Mode* in Wi-Fi networking?	A promiscuous mode where the interface captures all wireless traffic within range, regardless of intended recipient.	wifi

U7e1kF3pL2	Cloze (type in the answer)	Ops::WiFi Gym::Chain 2: Switching Interface Modes	Which sequence of commands sets <code>wlan0</code> into Managed mode and connects to ESSID HTB-Wifi?<br><br>{{c1::sudo ifconfig wlan0 down}}<br>{{c1::sudo iwconfig wlan0 mode managed}}<br>{{c1::sudo iwconfig wlan0 essid HTB-Wifi}}<br>{{c1::sudo ifconfig wlan0 up}}		wifi
T3q6mN9rK8	Cloze (type in the answer)	Ops::WiFi Gym::Chain 2: Switching Interface Modes	Which commands set <code>wlan0</code> into Ad-Hoc mode with ESSID HTB-Mesh?<br><br>{{c1::sudo iwconfig wlan0 mode ad-hoc}}<br>{{c1::sudo iwconfig wlan0 essid HTB-Mesh}}		wifi
S9b5dH2yQ4	Cloze (type in the answer)	Ops::WiFi Gym::Chain 2: Switching Interface Modes	What is the minimal <code>hostapd</code> configuration to start an open AP named HTB-Hello-World on channel 2?<br><br>{{c1::interface=wlan0}}<br>{{c1::driver=nl80211}}<br>{{c1::ssid=HTB-Hello-World}}<br>{{c1::channel=2}}<br>{{c1::hw_mode=g}}		wifi
R8k2fM4nL1	Basic (type in the answer)	Ops::WiFi Gym::Chain 2: Switching Interface Modes	What command launches an AP using <code>hostapd</code> with a configuration file named <code>open.conf</code>?	<code>sudo hostapd open.conf</code>	wifi
Q4n7xJ6rT0	Basic (type in the answer)	Ops::WiFi Gym::Chain 2: Switching Interface Modes	What command attempts to set <code>wlan0</code> into mesh mode?	<code>sudo iw dev wlan0 set type mesh</code>	wifi
P1m9zK5qV7	Cloze (type in the answer)	Ops::WiFi Gym::Chain 2: Switching Interface Modes	Which commands manually enable Monitor mode on <code>wlan0</code>?<br><br>{{c1::sudo ifconfig wlan0 down}}<br>{{c1::sudo iw wlan0 set monitor control}}<br>{{c1::sudo ifconfig wlan0 up}}		wifi
O6t2yD3pR8	Basic (type in the answer)	Ops::WiFi Gym::Chain 2: Switching Interface Modes	What command verifies the current mode of a Wi-Fi interface?	<code>iwconfig</code>	wifi
```