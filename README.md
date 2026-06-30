# KNOQ 

> Knock Network Open Query — port knock service detector for single hosts and subnet scanning.

![Shell](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat-square&logo=gnu-bash&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Kali-557C94?style=flat-square&logo=kalilinux&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)

---

## Overview

KNOQ automates **port knocking detection** against a target — sending a configured knock sequence and verifying whether a specified port opens in response. Supports both single-host testing and full `/24` subnet scanning with a parallel thread pool.

Designed for internal network assessments and authorized penetration tests.

---

## Features

- **Single host and subnet modes** — test one IP or scan an entire `/24` automatically
- **Fully configurable knock sequence** — any number of ports, any order, via `-k`
- **Configurable timing** — knock delay and post-knock wait time both adjustable
- **Parallel subnet scanning** — thread pool (default: 20) for fast network coverage
- **nmap + /dev/tcp fallback** — uses nmap for reliable port verification when available, falls back to native Bash otherwise
- **Banner grabbing** — attempts TCP banner and HTTP probe on the discovered port
- **Output to file** — saves findings with timestamped header
- **Silent mode** — findings only, suitable for piping

---

## Requirements

| Dependency | Purpose | Install |
|---|---|---|
| `bash 4.0+` | Script runtime | Pre-installed on Linux |
| `bc` | Knock delay calculation | `apt install bc` |
| `nmap` | Reliable port verification (optional) | `apt install nmap` |
| `nc` | Banner grabbing (optional) | `apt install netcat-openbsd` |

```bash
sudo apt install bc nmap netcat-openbsd
```

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/NeiveZ/KNOQ.git
cd KNOQ
```

### 2. Make the script executable

```bash
chmod +x knoq.sh
```

### 3. (Optional) Install globally

```bash
sudo cp knoq.sh /usr/local/bin/knoq
```

---

## Usage

```
./knoq.sh -t <target> -k <knock_ports> -p <target_port> [options]

Options:
  -t, --target        Single IP or subnet prefix (required)
                      Single:  192.168.1.10
                      Subnet:  192.168.1   (scans .1 to .254)
  -k, --knock         Knock sequence, comma-separated (required)
                      e.g. 1337,80,443,8080
  -p, --port          Target port to check after knock (required)
  -d, --knock-delay   Delay between knocks in ms (default: 100)
  -w, --wait          Seconds to wait after knock before checking port (default: 2)
  -T, --timeout       Connection timeout in seconds (default: 1)
  -t2, --threads      Parallel hosts in subnet mode (default: 20)
  -o, --output        Save findings to file
  --silent            Findings only — no per-host logs
  -h, --help          Show this help
```

---

## Examples

**Test a single host:**
```bash
./knoq.sh -t 192.168.1.10 -k 1337,80,443 -p 22
```

**Scan a full /24 subnet:**
```bash
./knoq.sh -t 192.168.1 -k 1337,80,443 -p 22
```

**Custom timing, save results:**
```bash
./knoq.sh -t 10.0.0 -k 500,1000,2000 -p 4444 -d 200 -w 3 -o findings.txt
```

**Higher parallelism on a fast network:**
```bash
./knoq.sh -t 172.16.0 -k 1337,80,443 -p 22 -t2 50
```

**Silent mode — pipe findings:**
```bash
./knoq.sh -t 192.168.1 -k 1337,80,443 -p 22 --silent | tee results.txt
```

---

## How Port Knocking Works

Port knocking is a method of externally opening ports by generating a connection attempt on a predefined sequence of closed ports. The firewall monitors connection attempts and opens the target port only when the exact sequence is received in order.

```
Client                        Server (firewall)
  |                               |
  |--- knock port 1337 ---------->|  (closed, logged)
  |--- knock port 80 ------------>|  (closed, logged)
  |--- knock port 443 ----------->|  (closed, logged)
  |                               |  sequence matched → opens port 22
  |--- connect port 22 ---------->|  (now open)
```

KNOQ automates this process across a network range to detect hosts running port knock daemons.

---

## Output

```
knoq | subnet 192.168.1.0/24 | knock: 1337,80,443 | target port: 22 | threads: 20

[23:10:04] Testing 192.168.1.1...
[23:10:04]   Knocking 192.168.1.1: 1337,80,443
[23:10:05]   Waiting 2s for port to activate...
[23:10:07]   Port 22 did not open on 192.168.1.1
...
[23:10:41][+] SERVICE FOUND at 192.168.1.42:22
  banner: SSH-2.0-OpenSSH_9.2p1

Scan complete | time: 48s
```

---

## Repository Structure

```
KNOQ/
└── knoq.sh    # Main script
```

---

## Legal

For use only on systems you own or have explicit written authorization to test.
Unauthorized use against third-party systems is illegal.
