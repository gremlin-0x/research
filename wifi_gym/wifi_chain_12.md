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