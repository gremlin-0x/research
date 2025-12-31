### 1.3 GPSD + virtual GPS + Kismet

```bash
# Stop socket-activated gpsd units and clean stale sockets (Debian/Ubuntu)
sudo systemctl stop gpsd gpsd.socket 2>/dev/null || true
sudo killall gpsd 2>/dev/null || true
sudo killall -9 gpsd 2>/dev/null || true
sudo rm -f /run/gpsd.sock
````

**What it does:**  
Cleans up any systemd-managed or stray `gpsd` instances and removes the default socket so you can run `gpsd` manually without conflicts.

**Command breakdown:**

- `sudo` – run each command with root privileges.
    
- `systemctl stop gpsd gpsd.socket` – stop the main service and its socket-activation unit (Debian/Ubuntu default).
    
- `killall gpsd` / `killall -9 gpsd` – make sure no background `gpsd` processes are left running.
    
- `rm -f /run/gpsd.sock` – remove any stale gpsd control socket so new clients can connect cleanly.
    

---

```bash
# Check raw NMEA output directly from the USB GPS (no gpsd running yet)
sudo gpsmon /dev/ttyUSB0
```

**What it does:**  
Reads **directly** from the serial GPS device so you can confirm it is actually emitting NMEA data before involving `gpsd`.

**Command breakdown:**

- `sudo` – access the serial device.
    
- `gpsmon` – ncurses GPS monitor.
    
- `/dev/ttyUSB0` – physical USB GPS serial device.
    

> **Note:** Use `gpsmon /dev/ttyUSB0` (or `/dev/pts/X`) to talk **to the device itself**. Once `gpsd` is bound to that device, switch to clients like `cgps` or `gpsmon` **without** a device argument to avoid two processes fighting over the same serial stream.

---

```bash
# Start gpsd on the USB GPS in the foreground (debug mode)
sudo gpsd -N -n -D 2 /dev/ttyUSB0
```

**What it does:**  
Runs `gpsd` attached to the USB GPS and keeps it in the foreground with basic debug output so you can see errors.

**Command breakdown:**

- `gpsd` – GPS daemon.
    
- `-N` – don’t daemonize; stay in the foreground (good for debugging).
    
- `-n` – start reading the device immediately instead of waiting for a client.
    
- `-D 2` – debug level 2 (moderate verbosity).
    
- `/dev/ttyUSB0` – serial device representing the USB GPS.
    

---

```bash
# Query gpsd using a curses client (after gpsd is running)
cgps -s
```

**What it does:**  
Connects to the running `gpsd` instance (via its socket) and shows current position, time, and satellite info.

**Command breakdown:**

- `cgps` – curses client for gpsd.
    
- `-s` – “simple” full-screen display mode.
    

> **Tip:** `gpsmon` **without arguments** also talks to `gpsd` via its socket. `gpsmon /dev/ttyUSB0` talks to the device directly; `gpsmon` (no args) talks to `gpsd`.

---

```bash
# Create two linked pseudo-terminals for a virtual GPS feed
socat -d -d pty,raw,echo=0 pty,raw,echo=0
```

**What it does:**  
Uses `socat` to create a **pair** of linked PTYs to simulate a GPS device and a data feeder.

**Command breakdown:**

- `socat` – stream redirection/relay tool.
    
- `-d -d` – print extra diagnostic/debug information twice (more verbose).
    
- `pty,raw,echo=0` – first pseudo-terminal in raw mode with echo off.
    
- `pty,raw,echo=0` – second pseudo-terminal linked to the first.
    

> **Important:** `socat` will print two lines like:  
> `PTY is /dev/pts/6`  
> `PTY is /dev/pts/7`  
> In this example, treat:
> 
> - `GPS_DEVICE=/dev/pts/6` – the “GPS receiver” side seen by gpsd.
>     
> - `FEED_PORT=/dev/pts/7` – the side you will write fake NMEA sentences to.
>     

---

```bash
# Start gpsd in the foreground on the simulated GPS device PTY
sudo gpsd -N -n -D 2 /dev/pts/6
```

**What it does:**  
Attaches `gpsd` to the **GPS_DEVICE** PTY from `socat` (here `/dev/pts/6`) in the foreground so you can see debug output.

**Command breakdown:**

- `-N` – don’t daemonize; keep output in this terminal.
    
- `-n` – read immediately, even without clients.
    
- `-D 2` – moderate debug level.
    
- `/dev/pts/6` – example PTY acting as the GPS device (`GPS_DEVICE`).
    

---

```bash
# Continuously feed test NMEA data into the other PTY (FEED_PORT)
while true; do
  printf '$GPRMC,123519,A,4807.038,N,01131.000,E,022.4,084.4,230394,003.1,W*6A\r\n' > /dev/pts/7
  sleep 1
done
```

**What it does:**  
Sends a synthetic `$GPRMC` NMEA sentence once per second into the **FEED_PORT** PTY so `gpsd` (on `/dev/pts/6`) sees a steady GPS stream.

**Command breakdown:**

- `while true; do ... done` – infinite loop.
    
- `printf '... \r\n'` – prints a single NMEA sentence with proper `\r\n` line ending.
    
- `> /dev/pts/7` – writes to the **FEED_PORT** PTY created by `socat`.
    
- `sleep 1` – wait one second between sentences.
    

> **HTB fix:** Some snippets mistakenly write to `/dev/pts/8` (or another PTY that doesn’t exist). Make sure you write to the **second PTY that `socat` printed** (e.g., `/dev/pts/7`), otherwise gpsd will never see any data.

---

```bash
# (Optional) Configure systemd-managed gpsd to always use the virtual PTY
sudo sed -i 's/DEVICES="\/dev\/pts\/[0-9]*"/DEVICES="\/dev\/pts\/6"/' /etc/default/gpsd
```

**What it does:**  
Updates `/etc/default/gpsd` so that when you later re-enable the systemd gpsd units, they will use `/dev/pts/6` as the GPS source.

**Command breakdown:**

- `sed -i` – edit the file **in place**.
    
- `DEVICES="\/dev\/pts\/[0-9]*"` – matches any existing `/dev/pts/N` device value.
    
- `DEVICES="\/dev\/pts\/6"` – replaces it with `/dev/pts/6`.
    
- `/etc/default/gpsd` – gpsd’s default configuration file.
    
