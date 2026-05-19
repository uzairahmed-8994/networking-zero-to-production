# Lab 02 — Reading Your Network

## What This Lab Covers

Before capturing traffic or diagnosing failures, the starting point
is always the same: read the current state of the system.

Three commands establish a complete picture of the network configuration:

- `ip addr` — interfaces, addresses, and link state
- `ip route` — kernel routing table and gateway decisions
- `ss -tulnp` — active sockets, listening services, owning processes

These are not diagnostic tools of last resort. They are the first thing
to run on any unfamiliar system. The output answers three foundational
questions: what addresses does this machine hold, where does traffic go,
and what is currently listening for connections.

---

## Prerequisites

Lab 01 completed. All tools installed and verified.

---

## The Analogy

Think of the machine as a building with multiple entrances.

`ip addr` identifies the entrances — each interface is a door, the IP
address is the street address on that door, the MAC address is the
physical serial number stamped on the door itself.

`ip route` is the internal mail routing system — rules that say "anything
addressed to 172.20.x.x goes out the east door, everything else goes
through reception who forwards it externally."

`ss -tulnp` is the room directory — room 53 has a DNS process waiting,
room 37677 has a node process waiting. Empty rooms do not appear.

---

## Notes

- All commands run inside WSL2 terminal
- Outputs shown here are real examples — actual values will differ
- Paste real output from the actual system being investigated

---

## Investigating Interface Configuration

### ip addr

```bash
ip addr
```

```
[paste real output here]
```

Starting with `ip addr` gives an immediate inventory of every network
interface the kernel has registered, along with its addressing and state.

A typical WSL2 system returns three interfaces. Each reveals something
specific about how this machine is positioned in the network.

---

**The loopback interface**

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet 10.255.255.254/32 brd 10.255.255.254 scope global lo
    inet6 ::1/128 scope host
```

The loopback interface is internal to the kernel. Traffic sent to
`127.0.0.1` never touches a physical or virtual NIC — the kernel
handles it entirely in software without routing it anywhere.

The `/8` prefix means the entire `127.0.0.0/8` range routes to loopback.
`127.0.0.1` is conventional, but `127.0.0.2` through `127.255.255.254`
are equally valid loopback addresses.

The second entry — `10.255.255.254/32` — is WSL2-specific. This is the
address of the internal DNS forwarder that WSL2 runs on the loopback
interface. The `/32` prefix means it is a host route: exactly one IP,
no subnet. This forwarder intercepts DNS queries and routes them to the
Windows resolver. 

---

**The primary Ethernet interface**

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP
    link/ether 00:15:5d:c3:7a:d2 brd ff:ff:ff:ff:ff:ff
    inet 172.20.193.120/20 brd 172.20.207.255 scope global eth0
    inet6 fe80::215:5dff:fec3:7ad2/64 scope link
```

`eth0` is the primary interface — the virtual NIC that Hyper-V presents
to the WSL2 instance. All external traffic moves through here.

The flags in angle brackets describe current state:

| Flag | Meaning |
|---|---|
| `BROADCAST` | Interface supports broadcast addressing |
| `MULTICAST` | Interface supports multicast |
| `UP` | Administratively enabled |
| `LOWER_UP` | Physical/virtual link is active |

`mtu 1500` is the Maximum Transmission Unit — the largest Ethernet frame
this interface will send without fragmentation. 1500 bytes is the standard
Ethernet MTU. MTU mismatches are a real production failure mode: a packet
larger than the path MTU gets dropped silently, causing connections to
hang. This value becomes directly relevant in the production failure
scenarios in Lab 22.

`link/ether 00:15:5d:c3:7a:d2` is the MAC address. This is the Layer 2
identity of the interface — the hardware-level address that Ethernet uses
for delivery within a local network segment. The `00:15:5d` prefix
identifies this as a Microsoft Hyper-V virtual NIC, which confirms the
WSL2 virtualization layer.

`brd ff:ff:ff:ff:ff:ff` is the Ethernet broadcast address. A frame
destined for this address is delivered to every device on the local
segment. ARP requests use this — when the system needs to find the MAC
address for an IP it does not yet know, it broadcasts a frame asking
"who has this IP?"

`inet 172.20.193.120/20` is the IPv4 address and prefix length.
The `/20` means the first 20 bits identify the network. This places
the interface in the `172.20.192.0/20` subnet, which spans
`172.20.192.0` through `172.20.207.255` — a range of 4096 addresses.
The broadcast address `172.20.207.255` confirms this.

`inet6 fe80::215:5dff:fec3:7ad2/64 scope link` is the link-local IPv6
address. Every interface gets one automatically derived from its MAC
address — no configuration required. `fe80::/10` is the link-local
prefix range. `scope link` means this address is only valid on this
local segment and will never be routed beyond it. This address is used
for neighbour discovery and router advertisements in IPv6 networks.

---

**The Docker bridge interface**

```
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN
    link/ether 22:ce:9e:1a:d9:3d brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
```

`docker0` is created automatically when Docker is installed. It is a
software bridge — a virtual Layer 2 switch implemented in the kernel.
When containers start, Docker creates a `veth` pair: one end is placed
inside the container's network namespace, the other is attached to
`docker0`. The container communicates through its end of the pair,
the host sees traffic arriving on `docker0`.

`NO-CARRIER` in the flags is significant. This means no interfaces are
currently attached to the bridge — no containers are running. When a
container starts, this flag disappears and the interface state moves
to `UP`. Checking for `NO-CARRIER` on `docker0` is a quick way to
confirm whether containers are running.

`172.17.0.1/16` is Docker's default bridge subnet. The host itself
holds `172.17.0.1` — this is the gateway address that running containers
use to reach external networks. The `/16` gives Docker a range of
65,534 usable container addresses (`172.17.0.2` through `172.17.255.254`).

This interface is explored in depth in the Docker networking lab.
The important observation here is that Docker's networking is visible
from the host as a standard kernel interface — it is not hidden inside
Docker's own abstraction.

---

**Additional ip addr commands**

```bash
# Show a single interface only
ip addr show eth0

# Show only IPv4 addresses
ip -4 addr show

# Verify the temporary IP appeared and disappeared
ip addr add 10.99.99.1/24 dev eth0
ip addr show eth0
ip addr del 10.99.99.1/24 dev eth0
```

Adding a temporary address to `eth0` and then removing it demonstrates
that IP addresses are kernel configuration, not fixed properties of the
interface. This is how cloud providers add and remove Elastic IPs from
EC2 instances — the same `ip addr add` and `ip addr del` operations
happen at the kernel level.

---

## Investigating the Routing Table

### ip route

```bash
ip route
```

```
[paste real output here]
```

The routing table contains the rules the kernel consults every time a
packet leaves this machine. The kernel reads the table top to bottom
and uses the first matching rule. Understanding this table is
prerequisite to understanding every routing failure.

---

**Route table analysis**

```
default via 172.20.192.1 dev eth0 proto kernel
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
172.20.192.0/20 dev eth0 proto kernel scope link src 172.20.193.120
```

**Default route:**

```
default via 172.20.192.1 dev eth0 proto kernel
```

`default` is shorthand for `0.0.0.0/0` — a route that matches every
possible destination. The kernel uses this when no more specific route
matches. `via 172.20.192.1` is the next-hop gateway: the Windows host
that WSL2 routes through. Every packet destined for the public internet
passes through this address.

This gateway is the boundary between WSL2's private network and the
outside world. The Windows host performs NAT here — it rewrites the
source IP from the WSL2 address (`172.20.193.120`) to the Windows
machine's public IP before forwarding. This is why external servers
see the Windows host's IP rather than the WSL2 IP.

Removing this route (`ip route del default`) would immediately break
all external connectivity — DNS queries would fail, HTTP connections
would fail, and ping to external IPs would fail. The machine would
still reach local addresses, but nothing beyond the local subnet.

**Docker's network route:**

```
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
```

This route tells the kernel to send any traffic destined for `172.17.x.x`
through `docker0`. `linkdown` indicates the bridge has no attached
interfaces — no containers are running. When a container starts, this
route activates and traffic to the container's IP is forwarded through
the bridge rather than through `eth0`.

The operational implication: if a container is assigned `172.17.0.2`,
the host can reach it directly via this route without NAT. Traffic stays
within the kernel's bridge implementation.

**Local subnet route:**

```
172.20.192.0/20 dev eth0 proto kernel scope link src 172.20.193.120
```

This route matches any address in the same subnet as `eth0`. Traffic
to these addresses goes directly through `eth0` without involving a
gateway — the destination is on the same local segment and can be reached
via ARP rather than routing.

`scope link` confirms this — the destination is link-local, reachable
directly without a router hop. `src 172.20.193.120` specifies which
source IP to use when sending from this interface.

---

**Querying a specific route decision**

```bash
ip route get 8.8.8.8
```

```
[paste output here]
```

`ip route get` asks the kernel to perform an actual route lookup for a
given destination and report which route it selected. The output shows
the interface, gateway, and source IP that would be used for that
specific destination. This is more precise than reading the route table
manually — it accounts for all the routing logic the kernel applies,
including longest-prefix matching.

The important observation here is that `ip route get` reflects the
actual decision the kernel makes, not just the configured rules.
In complex routing scenarios with multiple routes, this distinction
matters.

---

## Investigating Listening Sockets

### ss -tulnp

```bash
ss -tulnp
```

```
[paste real output here]
```

`ss` queries the kernel's socket table directly. The flags select
the scope: `-t` TCP, `-u` UDP, `-l` listening only, `-n` numeric
addresses and ports, `-p` process information.

A listening socket represents a service that has called `bind()` on
a port and `listen()` to accept connections. The kernel holds the
socket open and queues incoming connections until the process accepts them.

---

**Reading socket entries**

```
Netid  State   Recv-Q  Send-Q  Local Address:Port    Peer Address:Port  Process
tcp    LISTEN  0       1000    10.255.255.254:53      0.0.0.0:*
tcp    LISTEN  0       4096    127.0.0.53%lo:53       0.0.0.0:*
tcp    LISTEN  0       511     127.0.0.1:37677        0.0.0.0:*          users:(("node",pid=765))
udp    UNCONN  0       0       127.0.0.54:53          0.0.0.0:*
```

**Netid and State:**
TCP sockets in `LISTEN` state are waiting for incoming SYN packets.
UDP sockets show `UNCONN` — UDP has no connection state, so the socket
simply exists at a port, waiting for datagrams.

**Recv-Q and Send-Q:**
For listening TCP sockets, `Recv-Q` shows the number of connections
that have completed the three-way handshake but have not yet been
accepted by the application. `Send-Q` is the maximum backlog — how
many connections the kernel will queue. A `Recv-Q` value that is
consistently non-zero and growing indicates the application is not
accepting connections fast enough.

**Local Address interpretation:**

| Address | Scope |
|---|---|
| `0.0.0.0:80` | Listening on all interfaces — reachable from anywhere |
| `127.0.0.1:5432` | Loopback only — local processes only, not network-reachable |
| `10.255.255.254:53` | Specific interface — the WSL2 DNS forwarder |
| `:::80` | IPv6 equivalent of 0.0.0.0 |

The DNS entries on loopback addresses (`127.0.0.53`, `127.0.0.54`,
`10.255.255.254`) are the WSL2 DNS forwarding stack. These are the
resolvers that `cat /etc/resolv.conf` will point to. Queries arrive
here, the forwarder processes them, and forwards upstream to the
Windows DNS resolver.

The `node` process on port `37677` is a development server or IDE
helper. The port number is ephemeral — assigned by the kernel at
startup. The process name and PID identify it precisely.

---

**Filtering the socket list**

```bash
ss -tulnp | grep LISTEN
```

```
[paste output here]
```

This filters to TCP listeners only — a faster way to answer "what is
currently accepting connections on this machine" without reading
through UDP sockets.

---

## Observing Socket State Changes

Starting a process and watching its socket appear demonstrates that
`ss` reflects live kernel state, not a cached snapshot.

```bash
python3 -m http.server 8080 &
```

Immediately after:

```bash
ss -tulnp | grep 8080
```

```
[paste output here]
```

The socket appears within milliseconds of the process binding to the
port. The kernel updated the socket table as soon as the `bind()` and
`listen()` system calls completed.

Checking the routing for loopback traffic:

```bash
ip route get 127.0.0.1
```

```
[paste output here]
```

The routing table confirms that traffic to `127.0.0.1` routes via the
loopback interface — it never reaches `eth0`. A service bound to
`127.0.0.1:8080` is not reachable from the network, regardless of
firewall rules. The kernel routes the traffic internally before any
network stack processing occurs.

Stopping the server and verifying the socket is released:

```bash
kill %1
ss -tulnp | grep 8080
```

```
[paste output here — should be empty]
```

The socket disappears immediately after the process exits. If a socket
persisted after the process exited, it would indicate an abnormal
shutdown — a pattern worth recognising when investigating port binding
failures.

---

## Operational Summary

These three commands together establish the complete network state
of a system before any traffic analysis begins.

`ip addr` reveals interface configuration, addressing, and link state.
Docker interfaces, loopback addresses, and WSL2-specific DNS addresses
are all visible here.

`ip route` reveals the kernel's forwarding decisions. The default
gateway, container routing, and local subnet routing are all readable
without generating any traffic.

`ss -tulnp` reveals what is currently listening, on which addresses,
and which processes own those sockets.

In any connectivity investigation, these three commands run first.
They are the baseline. Everything captured with `tcpdump` or diagnosed
with `nc` is interpreted against the picture these three commands establish.

---

*Next: Lab 03 — Packet Capture Basics*
*tcpdump is used to capture live traffic. The output is saved to a pcap*
*file and opened in Wireshark for analysis.*