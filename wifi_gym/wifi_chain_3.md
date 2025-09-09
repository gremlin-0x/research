# Chain #3: Packet Capture & Analysis

This chain covers capturing Wi-Fi packets, analyzing raw 802.11 frames, and visualizing relationships between clients and access points.

---

## Step 1: Enable Monitor Mode

Before packet capture, the interface must be in **monitor mode** (so it sees all wireless traffic, not just what is addressed to it).

```bash
sudo airmon-ng start wlan0
```
- `airmon-ng start wlan0` → switches `wlan0` into monitor mode.
- This usually renames the interface to `wlan0mon`.
- `sudo iwconfig` can be used to confirm monitor mode is active.

---

## Step 2: Capture Packets with Airodump-ng

Start scanning and collecting raw 802.11 frames:

```bash
sudo airodump-ng wlan0mon
```
- Displays all nearby APs and clients.
- Each AP is listed with fields like:
  - **BSSID** → MAC address of AP.
  - **PWR** → signal strength.
  - **Beacons** → periodic announcements.
  - **ENC / CIPHER / AUTH** → encryption and authentication type.
  - **ESSID** → network name.
- Below the AP table is the **station table**, listing clients and which AP they’re connected to.

### Narrowing Down Capture

- Capture a specific **channel**:
  ```bash
  sudo airodump-ng -c 11 wlan0mon
  ```
- Capture multiple channels:
  ```bash
  sudo airodump-ng -c 1,6,11 wlan0mon
  ```
- Capture **5GHz band** instead of default 2.4GHz:
  ```bash
  sudo airodump-ng wlan0mon --band a
  ```
- Capture all bands (2.4GHz + 5GHz):
  ```bash
  sudo airodump-ng wlan0mon --band abg
  ```

---

## Step 3: Save Captured Data

Saving output ensures later processing (cracking, graphing, etc.):

```bash
sudo airodump-ng -w HTB wlan0mon
```
- `-w HTB` → writes capture files with prefix `HTB`.
- Files produced:
  - `HTB-01.cap` → full packet capture.
  - `HTB-01.csv` → AP/client details in CSV.
  - `HTB-01.kismet.csv` and `HTB-01.kismet.netxml` → formats for Kismet.
  - `HTB-01.log.csv` → log file.

---

## Step 4: Visualize Relationships with Airgraph-ng

`airgraph-ng` processes the CSV output of Airodump-ng into graphs.

### Generate Client-to-AP Relationship Graph (CAPR)

```bash
sudo airgraph-ng -i HTB-01.csv -g CAPR -o output.png
```
- `-i HTB-01.csv` → input CSV from Airodump-ng.
- `-g CAPR` → Clients-to-AP Relationship graph.
- `-o HTB_CAPR.png` → output graph image.
- AP color coding:
  - Green = WPA
  - Yellow = WEP
  - Red = Open
  - Black = Unknown

### Generate Common Probe Graph (CPG)

```bash
sudo airgraph-ng -i HTB-01.csv -g CPG -o HTB_CPG.png
```
- `-g CPG` → graph showing which SSIDs clients are probing for.
- Useful for discovering hidden SSIDs and client behavior.

---


## Anki Deck:

```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

M1x7aF9tL3	Basic (type in the answer)	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	Which command enables monitor mode on <code>wlan0</code> using Aircrack-ng tools?	<code>sudo airmon-ng start wlan0</code>	wifi
N4p2cJ8rT6	Basic (type in the answer)	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	How do you run Airodump-ng on <code>wlan0mon</code> to capture all wireless traffic?	<code>sudo airodump-ng wlan0mon</code>	wifi
O5m9vK6qR1	Basic (type in the answer)	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	How do you capture only on channel 11 with Airodump-ng?	<code>sudo airodump-ng -c 11 wlan0mon</code>	wifi
P2s8xH4nL7	Basic (type in the answer)	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	Which command captures on multiple channels (1, 6, 11)?	<code>sudo airodump-ng -c 1,6,11 wlan0mon</code>	wifi
Q0z3bD7pV8	Basic (type in the answer)	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	Which command captures only 5GHz networks with Airodump-ng?	<code>sudo airodump-ng wlan0mon --band a</code>	wifi
R6k4nJ2mT9	Basic (type in the answer)	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	How do you scan across all Wi-Fi bands (2.4GHz + 5GHz) with Airodump-ng?	<code>sudo airodump-ng wlan0mon --band abg</code>	wifi
S8y1fE5qM2	Basic (type in the answer)	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	Which command saves Airodump-ng capture files with prefix HTB?	<code>sudo airodump-ng -w HTB wlan0mon</code>	wifi
T3u7dK9rP4	Basic (type in the answer)	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	After running Airodump-ng and saving a CSV (e.g. <code>HTB-01.csv</code>), which command generates a Clients-to-AP Relationship Graph (CAPR) and writes it to a <code>output.png</code> file?	<code>sudo airgraph-ng -i HTB-01.csv -g CAPR -o output.png</code>	wifi
U9o5hL6qR3	Basic (type in the answer)	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	After running Airodump-ng and saving a CSV (e.g. <code>HTB-01.csv</code>), which command generates a Common Probe Graph (CPG) and writes it to a PNG file (e.g. <code>HTB_CPG.png</code>)?	<code>sudo airgraph-ng -i HTB-01.csv -g CPG -o HTB_CPG.png</code>	wifi
V2e6nF7tL1	Basic	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	What does monitor mode allow a Wi-Fi interface to do?	Capture all wireless packets within range, regardless of destination.	wifi
W5m0xJ8rQ2	Basic	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	In Airodump-ng output, what does <b>BSSID</b> stand for?	The MAC address of the access point.	wifi
X3t4bH9qP7	Basic	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	In Airodump-ng output, what does <b>PWR</b> indicate?	Signal strength of the network (higher = stronger).	wifi
Y1p9dK6mR5	Basic	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	In Airodump-ng output, what are <b>Beacons</b>?	Announcement packets sent by the AP to advertise its presence.	wifi
Z8q7fM2nL6	Basic	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	In Airodump-ng output, what does <b>ENC</b> mean?	Encryption method used (e.g., WEP, WPA2).	wifi
A4r1cE9tQ8	Basic	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	In Airodump-ng output, what does <b>CIPHER</b> indicate?	The encryption algorithm (e.g., TKIP, CCMP).	wifi
B0n5vK7pM3	Basic	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	In Airodump-ng output, what does <b>AUTH</b> mean?	The authentication method (e.g., PSK, SAE).	wifi
C6m2xH4qR9	Basic	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	What does the <b>station table</b> in Airodump-ng show?	Clients (stations) connected to APs, including their MAC addresses and activity.	wifi
D9o8jL1tQ2	Basic	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	What does Airgraph-ng do with Airodump-ng CSV files?	Generates visual graphs of AP-client relationships and client probe requests.	wifi
E7k3fM5nP1	Basic	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	In Airgraph-ng CAPR graph, what color represents an open network?	Red.	wifi
F2t6bH8qL4	Basic	Ops::WiFi Gym::Chain 3: Packet Capture & Analysis	In Airgraph-ng CAPR graph, what color represents a WPA-protected network?	Green.	wifi
```