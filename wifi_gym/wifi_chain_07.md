# Chain #7: Connecting to Wi-Fi Networks

This chain covers how to connect to Wi-Fi networks after credentials have been obtained. It includes both GUI and CLI workflows, with diversions for WEP, WPA/WPA2 Personal, WPA3, and WPA/WPA2 Enterprise.

---

## Step 1: Identify Available Networks

**Command:**
```bash
sudo iwlist wlan0 scan | grep 'Cell\|Quality\|ESSID\|IEEE'
```

**Explanation:**
- `iwlist wlan0 scan` → scans for all available Wi-Fi networks on interface `wlan0`.
- `grep` filter → shows only useful details (Cell number, Quality, ESSID, IEEE info).

**Output Example:**
- You might see networks with different encryption types: WEP, WPA/WPA2, WPA3, Enterprise.

**Diversion:**
- If your interface is in **monitor mode**, switch it back to **managed mode** before connecting.
```bash
sudo ifconfig wlan0 down
sudo iwconfig wlan0 mode managed
sudo ifconfig wlan0 up
```

---

## Step 2: Connect Depending on Security Type

### Case A: WEP Network

1. Create a config file `wep.conf`:
   ```
   network={
       ssid="HackTheBox"
       key_mgmt=NONE
       wep_key0=3C1C3A3BAB
       wep_tx_keyidx=0
   }
   ```

   - `ssid` → name of the target network.
   - `key_mgmt=NONE` → means no WPA, only WEP or open.
   - `wep_key0` → hex key for WEP.
   - `wep_tx_keyidx=0` → index of the WEP key slot.

2. Run wpa_supplicant:
   ```bash
   sudo wpa_supplicant -c wep.conf -i wlan0
   ```

3. Obtain IP:
   ```bash
   sudo dhclient wlan0
   ```

---

### Case B: WPA/WPA2 Personal Network

1. Create config file `wpa.conf`:
   ```
   network={
       ssid="HackMe"
       psk="password123"
   }
   ```

2. Connect with wpa_supplicant:
   ```bash
   sudo wpa_supplicant -c wpa.conf -i wlan0
   ```

3. Release old IP (if needed):
   ```bash
   sudo dhclient wlan0 -r
   ```

4. Request new IP:
   ```bash
   sudo dhclient wlan0
   ```

**Diversion:** WPA3 requires **SAE** instead of PSK.
```
network={
    ssid="MyWPA3"
    psk="password123"
    key_mgmt=SAE
}
```

---

### Case C: WPA/WPA2 Enterprise

1. Create config file `enterprise.conf`:
   ```
   network={
       ssid="HTB-Corp"
       key_mgmt=WPA-EAP
       identity="HTB\\Administrator"
       password="Admin@123"
   }
   ```

   - `key_mgmt=WPA-EAP` → indicates Enterprise auth.
   - `identity` → username.
   - `password` → password.

2. Connect:
   ```bash
   sudo wpa_supplicant -c enterprise.conf -i wlan0
   ```

3. Request IP:
   ```bash
   sudo dhclient wlan0
   ```

---

## Step 3: Alternative — Network Manager CLI (if available)

```bash
sudo nmtui
```
- Provides a text-based UI.
- Choose **Activate a connection** → select Wi-Fi → enter password.

---

## Anki Deck:

```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6
c1tP7aQzGJ	Basic	Ops::WiFi Gym::Chain 07: Connecting to Wi-Fi Networks	What does <code>key_mgmt=NONE</code> mean in a <code>wpa_supplicant</code> config?	It specifies no WPA authentication, used for WEP or open networks.	wifi
Sg3kzX2D8a	Basic	Ops::WiFi Gym::Chain 07: Connecting to Wi-Fi Networks	What is <code>SAE</code> in Wi-Fi authentication?	Simultaneous Authentication of Equals, the WPA3 key exchange method replacing PSK.	wifi
Qv9hL2kDfE	Basic	Ops::WiFi Gym::Chain 07: Connecting to Wi-Fi Networks	What is the purpose of <code>wpa_supplicant</code>?	It manages Wi-Fi authentication and association to networks from the CLI using config files.	wifi
Nr7eZx0iT4	Basic	Ops::WiFi Gym::Chain 07: Connecting to Wi-Fi Networks	What is the purpose of <code>dhclient</code> in Wi-Fi connections?	It requests or releases an IP address from the network’s DHCP server after authentication.	wifi
f6Uo0QvL8Y	Basic (type in the answer)	Ops::WiFi Gym::Chain 07: Connecting to Wi-Fi Networks	Which command scans for Wi-Fi networks and shows only <code>Cell</code>, <code>Quality</code>, <code>ESSID</code>, and <code>IEEE</code> fields?	<code>sudo iwlist wlan0 scan | grep 'Cell\|Quality\|ESSID\|IEEE'</code>	wifi
v5Yt2MnHgR	Cloze (type in the answer)	Ops::WiFi Gym::Chain 07: Connecting to Wi-Fi Networks	Which three commands switch interface <code>wlan0</code> back to managed mode?<br><br>{{c1::sudo ifconfig wlan0 down}}<br>{{c1::sudo iwconfig wlan0 mode managed}}<br>{{c1::sudo ifconfig wlan0 up}}		wifi
Kj8rZ1yBxP	Basic (type in the answer)	Ops::WiFi Gym::Chain 07: Connecting to Wi-Fi Networks	Which <code>wpa_supplicant</code> command connects interface <code>wlan0</code> to a WEP network using config file <code>wep.conf</code>?	<code>sudo wpa_supplicant -c wep.conf -i wlan0</code>	wifi
Lp4dX8qRnC	Basic (type in the answer)	Ops::WiFi Gym::Chain 07: Connecting to Wi-Fi Networks	Which <code>wpa_supplicant</code> command connects interface <code>wlan0</code> to a WPA/WPA2 PSK network using config file <code>wpa.conf</code>?	<code>sudo wpa_supplicant -c wpa.conf -i wlan0</code>	wifi
Tr1sJ6fVhE	Basic (type in the answer)	Ops::WiFi Gym::Chain 07: Connecting to Wi-Fi Networks	Which <code>wpa_supplicant</code> command connects interface <code>wlan0</code> to a WPA/WPA2 Enterprise network using config file <code>enterprise.conf</code>?	<code>sudo wpa_supplicant -c enterprise.conf -i wlan0</code>	wifi
Bm9tK4jWxF	Basic (type in the answer)	Ops::WiFi Gym::Chain 07: Connecting to Wi-Fi Networks	Which <code>dhclient</code> command releases the DHCP lease on interface <code>wlan0</code>?	<code>sudo dhclient wlan0 -r</code>	wifi
Hy3nC0bUsQ	Basic (type in the answer)	Ops::WiFi Gym::Chain 07: Connecting to Wi-Fi Networks	Which <code>dhclient</code> command requests a new DHCP lease on interface <code>wlan0</code>	<code>sudo dhclient wlan0</code>	wifi
Zo6pE2vDdM	Basic (type in the answer)	Ops::WiFi Gym::Chain 07: Connecting to Wi-Fi Networks	Which command launches the text-based UI tool <code>nmtui</code> for managing Wi-Fi connections?	<code>sudo nmtui</code>	wifi
```