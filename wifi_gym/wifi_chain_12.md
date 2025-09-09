# Chain #12: CPU vs GPU Cracking & Performance Tuning

## Goal & Scope

Understand how hardware impacts WPA password cracking speed and how to optimize Hashcat’s performance:

- Difference between CPU and GPU cracking.
    
- Measuring hash rate.
    
- Identifying available devices.
    
- Selecting CPU/GPU.
    
- Workload tuning (`-w`).
    
- Optimized kernels (`-O`).
    

---

## Step 1 — Why Speed Matters

WPA cracking often involves huge keyspaces. Efficiency depends on **hash rate** (password guesses per second).

- Higher hash rate = faster cracking.
    
- GPU parallelism usually outperforms CPU sequential logic.
    

---

## Step 2 — CPU Cracking

Characteristics:

- 2–16 cores typical.
    
- Optimized for sequential tasks.
    
- Can crack WPA but slower.
    

**Command:**

```bash
hashcat -m 22000 hash wordlist.txt -D 1
```

- `-D 1`: Use CPU only.
    

Bind to specific cores with affinity:

```bash
hashcat -m 22000 hash wordlist.txt -D 1 --cpu-affinity=1,2,3,4
```

- `--cpu-affinity`: Restrict cracking to selected cores.
    

---

## Step 3 — GPU Cracking

Characteristics:

- Hundreds to thousands of cores.
    
- Ideal for parallel hash computation.
    

**Command:**

```bash
hashcat -m 22000 hash wordlist.txt -D 2
```

- `-D 2`: Use GPU.
    

Enable optimized kernels:

```bash
hashcat -m 22000 hash wordlist.txt -D 2 -O
```

- `-O`: Enable optimized kernels → faster cracking (limitations: max password length may shrink).
    

---

## Step 4 — CPU + GPU Together

Hashcat can use multiple devices:

```bash
hashcat -m 22000 hash wordlist.txt -D 1,2
```

- `-D 1,2`: Use both CPU and GPU.
    
- Combine flexibility of CPU with GPU speed.
    

---

## Step 5 — Identify Devices

List all detected devices:

```bash
hashcat -I
```

- Displays OpenCL backends, device IDs, memory, cores.
    

Select a specific device by ID:

```bash
hashcat -m 22000 hash wordlist.txt -D 2 -d 2
```

- `-d 2`: Use device ID #2 (from `-I` output).
    

---

## Step 6 — Workload Profiles

Control power vs performance with `-w`:

- `-w 1`: Low → minimal system impact.
    
- `-w 2`: Default.
    
- `-w 3`: High.
    
- `-w 4`: Nightmare → max performance, heavy load.
    

Example:

```bash
hashcat -m 22000 hash wordlist.txt -D 2 -w 3
```

---

## Step 7 — Practical Notes

- GPUs dominate for WPA cracking.
    
- CPUs useful when GPU not available or for special hashes.
    
- Workload tuning must consider cooling and system stability.
    
- Optimized kernels (-O) give speed boost but may limit max length.
    

---

## Checklist (Quick Run)

1. List devices with `hashcat -I`.
    
2. Choose CPU (`-D 1`), GPU (`-D 2`), or both (`-D 1,2`).
    
3. Optionally target device ID with `-d`.
    
4. Increase performance with `-O`.
    
5. Adjust load with `-w 1–4`.
    

---

## Anki Deck:

```
#separator:tab
#html:true
#guid column:1
#notetype column:2
#deck column:3
#tags column:6

P7d2K9m4X1	Basic	Ops::WiFi Gym::Chain 12: CPU vs GPU Cracking	What does hash rate represent in WPA cracking?	The number of password guesses (hash computations) per second.	wifi
R4g7N2b8L5	Basic	Ops::WiFi Gym::Chain 12: CPU vs GPU Cracking	Why are GPUs generally faster than CPUs for password cracking?	Because GPUs have thousands of cores optimized for parallel computation, while CPUs have fewer cores optimized for sequential tasks.	wifi
S9k1H3q6M2	Basic (type in the answer)	Ops::WiFi Gym::Chain 12: CPU vs GPU Cracking	What Hashcat flag restricts cracking to CPU only?	<code>-D 1</code>	wifi
T6j3L8n1C7	Basic (type in the answer)	Ops::WiFi Gym::Chain 12: CPU vs GPU Cracking	What Hashcat flag restricts cracking to GPU only?	<code>-D 2</code>	wifi
U2m5P9r4B8	Basic	Ops::WiFi Gym::Chain 12: CPU vs GPU Cracking	What does the flag <code>-O</code> do in Hashcat?	Enables optimized kernels for faster cracking but may restrict maximum password length.	wifi
V1c4K7d9Q3	Basic	Ops::WiFi Gym::Chain 12: CPU vs GPU Cracking	What is the effect of <code>--cpu-affinity</code> in Hashcat?	It binds cracking threads to specific CPU cores to control workload distribution.	wifi
W3e6N8p2Z4	Basic (type in the answer)	Ops::WiFi Gym::Chain 12: CPU vs GPU Cracking	What Hashcat command cracks a WPA hash with CPU only, using cores 1–4?	<code>hashcat -m 22000 hash wordlist.txt -D 1 --cpu-affinity=1,2,3,4</code>	wifi
X5h9Q2s7J1	Basic	Ops::WiFi Gym::Chain 12: CPU vs GPU Cracking	How do you list all available cracking devices in Hashcat?	<code>hashcat -I</code>	wifi
Y8a3K6t5L9	Basic (type in the answer)	Ops::WiFi Gym::Chain 12: CPU vs GPU Cracking	What command selects device ID #2 on GPU only?	<code>hashcat -m 22000 hash wordlist.txt -D 2 -d 2</code>	wifi
Z7f2M4b6V8	Basic	Ops::WiFi Gym::Chain 12: CPU vs GPU Cracking	What are the four workload levels in Hashcat and what does -w 4 mean?	-w 1 low, -w 2 default, -w 3 high, -w 4 nightmare; -w 4 = max performance with heavy system load.	wifi
A9k5L1r3C2	Basic (type in the answer)	Ops::WiFi Gym::Chain 12: CPU vs GPU Cracking	Show a command that uses both CPU and GPU together on a WPA hash.	<code>hashcat -m 22000 hash wordlist.txt -D 1,2</code>	wifi
B6p8N9d2F4	Basic	Ops::WiFi Gym::Chain 12: CPU vs GPU Cracking	What practical risks come with running Hashcat at -w 4 (Nightmare)?	High power consumption, overheating, and potential system instability.	wifi
C4m7H2q9J5	Basic	Ops::WiFi Gym::Chain 12: CPU vs GPU Cracking	Why might a CPU still be useful for cracking despite being slower?	It can handle certain hash algorithms better, is available when GPU isn’t, or can supplement GPU cracking.	wifi
```