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