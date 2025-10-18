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