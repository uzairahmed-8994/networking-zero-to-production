# Lab 01 — Tools Setup and Verification

## What This Lab Does

Before any practical networking work, every tool must be installed and
confirmed working. Nothing is more frustrating than being mid-lab and
discovering a tool is missing or broken.

This lab installs everything once and verifies each tool with one simple
command. No deep usage yet. Just confirm it works.

---

## Prerequisites

Lab 00 completed. You understand what layer each tool operates at.

---

## Notes

- All commands run inside WSL2 terminal
- Wireshark is the only tool that installs on Windows, not WSL2
- Run every command even if you think the tool is already installed

---

## Why These Tools Matter

You are not installing random utilities.

Each tool solves a different type of networking problem:

| Tool | What You Will Eventually Use It For |
|---|---|
| `tcpdump` | Capture and inspect raw packets |
| `Wireshark` / `tshark` | Visual protocol analysis |
| `ss` | Inspect sockets and connection states |
| `dig` | Diagnose DNS problems |
| `nc` (netcat) | Test raw TCP/UDP connectivity |
| `curl` | Debug HTTP/HTTPS requests |
| `openssl` | Diagnose TLS and certificate issues |
| `ip` | Inspect interfaces, addresses, and routing |
| `traceroute` / `mtr` | Trace packet paths across networks |
| `conntrack` | Inspect connection tracking state |
| `iperf3` | Measure network throughput and performance |

Throughout this repo, you will combine multiple tools together to diagnose real networking failures.

---

## Step 1 — Install All Tools

Run this single command to install everything at once:

```bash
sudo apt update && sudo apt install -y \
  tcpdump \
  tshark \
  nmap \
  netcat-openbsd \
  socat \
  mtr \
  ipcalc \
  conntrack \
  dnsutils \
  iperf3 \
  iftop \
  nethogs \
  net-tools \
  traceroute \
  curl \
  wget \
  openssl
```

This will take 1-2 minutes. Let it complete fully.

---

## Step 2 — Verify Each Tool

Run each command below. You are only checking that the tool responds.
You do not need to understand the output yet.

**ip**
```bash
ip -V
```
Expected: a version number like `ip utility, iproute2-6.x.x`

---

**ss**
```bash
ss --version
```
Expected: a version number

---

**tcpdump**
```bash
sudo tcpdump --version
```
Expected: `tcpdump version x.x.x`

---

**tshark**
```bash
tshark --version
```
Expected: `TShark (Wireshark) x.x.x`

---

**ping**
```bash
ping -c 1 8.8.8.8
```
Expected: one line like `64 bytes from 8.8.8.8: icmp_seq=1 ttl=xxx time=xx ms`

---

**traceroute**
```bash
traceroute --version
```
Expected: version output

---

**mtr**
```bash
mtr --version
```
Expected: `mtr x.x.x`

---

**nc (netcat)**
```bash
nc -zv google.com 443
```
Expected: `connection succeeded message for TCP port 443`

---

**nmap**
```bash
nmap --version
```
Expected: `Nmap version x.x.x`

---

**dig**
```bash
dig -v
```
Expected: `DiG x.x.x`

---

**curl**
```bash
curl --version
```
Expected: `curl x.x.x (x86_64-pc-linux-gnu)`

---

**openssl**
```bash
openssl version
```
Expected: `OpenSSL x.x.x`

---

**iperf3**
```bash
iperf3 --version
```
Expected: `iperf 3.x.x`

---

**ipcalc**
```bash
ipcalc 192.168.1.0/24
```
Expected: a table showing network address, broadcast, netmask

---

**iftop**
```bash
which iftop
```
Expected: `/usr/sbin/iftop`

---

**nethogs**
```bash
which nethogs
```
Expected: `/usr/sbin/nethogs`

---

## Step 3 — Install Wireshark on Windows

Wireshark runs on Windows and captures WSL2 traffic through the
`vEthernet (WSL)` adapter.

If Wireshark is not installed yet:
1. Go to https://www.wireshark.org/download.html
2. Download Windows x64 installer
3. Run it — keep Npcap checked when asked
4. Open Wireshark, find `vEthernet (WSL)` in the interface list
5. Double click it to start a capture
6. In WSL2 run: `ping -c 3 8.8.8.8`
7. You should see ICMP packets appear in Wireshark

---

## Step 4 — Confirm tshark Works Inside WSL2

tshark is the terminal version of Wireshark. This is extremely useful on remote Linux servers where no graphical interface exists. It runs inside WSL2.
You will use it when you want Wireshark-style output without switching windows.

```bash
sudo tshark -i eth0 -c 5
```

Then in a second terminal:
```bash
ping -c 5 8.8.8.8
```

You should see 5 packets captured in tshark output. Press Ctrl+C if it
does not stop automatically.

---

## What I Saw

### Tools installed successfully
```
All tools listed above are installed successfully
```

### tshark verification output
```
Running as user "root" and group "root". This could be dangerous.
Capturing on 'eth0'
    1 0.000000000 172.20.193.120 → 8.8.8.8      ICMP 98 Echo (ping) request  id=0x9fb3, seq=1/256, ttl=64
    2 0.032198001      8.8.8.8 → 172.20.193.120 ICMP 98 Echo (ping) reply    id=0x9fb3, seq=1/256, ttl=117 (request in 1)
    3 1.022775137 172.20.193.120 → 8.8.8.8      ICMP 98 Echo (ping) request  id=0x9fb3, seq=2/512, ttl=64
    4 1.048194838      8.8.8.8 → 172.20.193.120 ICMP 98 Echo (ping) reply    id=0x9fb3, seq=2/512, ttl=117 (request in 3)
    5 2.045613763 172.20.193.120 → 8.8.8.8      ICMP 98 Echo (ping) request  id=0x9fb3, seq=3/768, ttl=64
5 packets captured
```

### Any tools that did not install or verify correctly
```
All tools installed successfully
```

---

## After This Lab

Every tool used in this repo is installed and verified.
You know each tool by name and which layer it operates at from Lab 00.
Nothing will block you mid-lab because of a missing tool.

---

*Next: Lab 02 — Reading Your Network*
*Three commands. Full understanding of your machine's current network state.*
