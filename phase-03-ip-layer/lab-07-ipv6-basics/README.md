# Lab 07 — IPv6 Basics

## What This Lab Covers

Since Lab 02, every `ip addr` output has included a line like this:

```
inet6 fe80::215:5dff:fec3:7ad2/64 scope link
```

It was never explained. This lab explains it.

IPv6 is not a replacement for IPv4 that requires learning everything
from scratch. It is the same addressing concept — unique identifiers
for network interfaces with a larger address space and a different
notation.

The goal here is not mastery. It is enough familiarity to read IPv6
addresses, understand the types, and not be blocked when IPv6 appears
in production environments, Kubernetes clusters, or AWS.

---

## Prerequisites

Lab 06 completed. IP addressing, subnet prefixes, and CIDR notation
are understood for IPv4.

---

## Why IPv6 Exists

IPv4 addresses are 32 bits — approximately 4.3 billion total addresses.
Public IPv4 space is heavily exhausted, which is why NAT and private
addressing became necessary at internet scale.
Large regions of the internet (especially in
Asia and mobile networks) primarily use IPv6.

IPv6 addresses are 128 bits — approximately 340 undecillion addresses.
Every device that will ever exist can have a globally unique address
without NAT.

The practical consequence: AWS services, Kubernetes clusters, and modern
Linux distributions run dual-stack — both IPv4 and IPv6 simultaneously.
Ignoring IPv6 entirely is no longer realistic.

---

## IPv6 Address Format

IPv4 wrote 32 bits as four decimal groups: `192.168.1.100`

IPv6 writes 128 bits as eight hexadecimal groups separated by colons:

```
Full form:    2001:0db8:85a3:0000:0000:8a2e:0370:7334
Compressed:   2001:db8:85a3::8a2e:370:7334
```

Two compression rules reduce the verbosity:

**Leading zeros in a group are omitted:**
`0db8` → `db8`, `0000` → `0`, `0370` → `370`

**One consecutive sequence of all-zero groups is replaced with `::`:**
`0000:0000` → `::`

The `::` can only appear once in an address. Its position determines
how many zero groups it represents.

```
::1              → 0000:0000:0000:0000:0000:0000:0000:0001  (loopback)
fe80::215:5dff   → fe80:0000:0000:0000:0215:5dff:...
```

---

## IPv6 Address Types

Three types are relevant for daily DevOps work:

**Loopback — `::1`**
The IPv6 equivalent of `127.0.0.1`. Traffic sent to `::1` never leaves
the machine. Always present on every system.

**Link-local — `fe80::/10`**
Automatically configured on every IPv6-capable interface. In many environments, parts of the link-local address are derived from the interface MAC address or generated from interface-specific data, which is why
`fe80::215:5dff:fec3:7ad2` contains fragments of `00:15:5d:c3:7a:d2`.
Link-local addresses are not routable — they only work on the local
network segment. They appear in routing protocols and neighbour
discovery but are never used for internet traffic.

**Global unicast — `2001::/32` and similar**
Routable across the internet. The IPv6 equivalent of a public IPv4
address. Assigned by the network provider. Not all environments
(including some WSL2 setups) have a global unicast address.


---

## Inspecting IPv6 Addresses

```bash
ip -6 addr show
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 state UNKNOWN qlen 1000
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 state UP qlen 1000
    inet6 fe80::215:5dff:fec3:7ad2/64 scope link 
       valid_lft forever preferred_lft forever
```

The `-6` flag filters to IPv6 only. The same information appears in
the regular `ip addr` output but the `-6` flag removes the IPv4 lines
for clarity.

Two entries will appear on most systems:

```
1: lo    inet6 ::1/128 scope host
2: eth0  inet6 fe80::215:5dff:fec3:7ad2/64 scope link
```

`::1/128` on loopback — the `/128` means all 128 bits identify this
specific address. There is no subnet, only one address.

`fe80::.../64` on eth0 — the `/64` is the standard IPv6 subnet prefix.
`scope link` confirms this is link-local and not routable.

A global unicast address would show `scope global` instead of
`scope link`. If the environment has one, it appears here.

---

## Pinging Over IPv6

```bash
ping6 ::1 -c 3
```

```
PING ::1 (::1) 56 data bytes
64 bytes from ::1: icmp_seq=1 ttl=64 time=0.682 ms
64 bytes from ::1: icmp_seq=2 ttl=64 time=0.040 ms
64 bytes from ::1: icmp_seq=3 ttl=64 time=0.048 ms

--- ::1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2094ms
rtt min/avg/max/mdev = 0.040/0.256/0.682/0.300 ms
```

This confirms the IPv6 stack is functional. The loopback address always
responds regardless of network configuration.

Pinging the link-local address requires specifying which interface to
use. This is the `%interface` scope notation:

```bash
ping6 fe80::215:5dff:fec3:7ad2%eth0 -c 3
```

```
PING fe80::215:5dff:fec3:7ad2%eth0 (fe80::215:5dff:fec3:7ad2%eth0) 56 data bytes
64 bytes from fe80::215:5dff:fec3:7ad2%eth0: icmp_seq=1 ttl=64 time=0.226 ms
64 bytes from fe80::215:5dff:fec3:7ad2%eth0: icmp_seq=2 ttl=64 time=0.041 ms
64 bytes from fe80::215:5dff:fec3:7ad2%eth0: icmp_seq=3 ttl=64 time=0.034 ms

--- fe80::215:5dff:fec3:7ad2%eth0 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2039ms
rtt min/avg/max/mdev = 0.034/0.100/0.226/0.088 ms
```

The `%eth0` suffix is required because link-local addresses are not
unique across the machine — two different interfaces could theoretically
have the same link-local address. The interface scope resolves the
ambiguity. Without it, the kernel does not know which interface to
use and returns an error.

---

## IPv6 Routing Table

The IPv6 routing table uses the same basic structure as IPv4 routing:
routes point traffic toward interfaces or gateways.

In this WSL2 environment, only one IPv6 route appears:

```bash
ip -6 route show
```

```
fe80::/64 dev eth0 proto kernel metric 256 pref medium
```
Read the important parts first:

- `fe80::/64` → the link-local IPv6 subnet
- `dev eth0` → traffic leaves through eth0
- `proto kernel` → route created automatically by the kernel
- `metric 256` → route preference value

No default IPv6 route exists in this environment because WSL2 is not
currently configured for global IPv6 internet connectivity.

This means:
- local IPv6 communication works
- IPv6 internet routing does not


---

## DNS and IPv6 — AAAA Records

IPv4 addresses are stored in DNS as A records. IPv6 addresses are
stored as AAAA records (four As, because IPv6 addresses are four times
the size of IPv4).

```bash
dig AAAA google.com
```

```
; <<>> DiG 9.18.39-0ubuntu0.24.04.3-Ubuntu <<>> AAAA google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45222
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 4, ADDITIONAL: 9

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;google.com.                    IN      AAAA

;; ANSWER SECTION:
google.com.             199     IN      AAAA    2a00:1450:4018:813::200e

;; AUTHORITY SECTION:
google.com.             51301   IN      NS      ns4.google.com.
google.com.             51301   IN      NS      ns3.google.com.
google.com.             51301   IN      NS      ns1.google.com.
google.com.             51301   IN      NS      ns2.google.com.

;; ADDITIONAL SECTION:
ns1.google.com.         51499   IN      A       216.239.32.10
ns2.google.com.         51499   IN      A       216.239.34.10
ns3.google.com.         51499   IN      A       216.239.36.10
ns4.google.com.         51499   IN      A       216.239.38.10
ns1.google.com.         51499   IN      AAAA    2001:4860:4802:32::a
ns2.google.com.         51499   IN      AAAA    2001:4860:4802:34::a
ns3.google.com.         51499   IN      AAAA    2001:4860:4802:36::a
ns4.google.com.         51499   IN      AAAA    2001:4860:4802:38::a

;; Query time: 19 msec
;; SERVER: 10.255.255.254#53(10.255.255.254) (UDP)
;; WHEN: Fri May 22 18:46:20 PKT 2026
;; MSG SIZE  rcvd: 315
```

The ANSWER section contains one or more AAAA records with full IPv6
addresses. Compare to the A record:

```bash
dig A google.com
```

```
; <<>> DiG 9.18.39-0ubuntu0.24.04.3-Ubuntu <<>> A google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11845
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 4, ADDITIONAL: 9

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             176     IN      A       142.251.38.14

;; AUTHORITY SECTION:
google.com.             51273   IN      NS      ns4.google.com.
google.com.             51273   IN      NS      ns2.google.com.
google.com.             51273   IN      NS      ns1.google.com.
google.com.             51273   IN      NS      ns3.google.com.

;; ADDITIONAL SECTION:
ns1.google.com.         51471   IN      A       216.239.32.10
ns2.google.com.         51471   IN      A       216.239.34.10
ns3.google.com.         51471   IN      A       216.239.36.10
ns4.google.com.         51471   IN      A       216.239.38.10
ns1.google.com.         51471   IN      AAAA    2001:4860:4802:32::a
ns2.google.com.         51471   IN      AAAA    2001:4860:4802:34::a
ns3.google.com.         51471   IN      AAAA    2001:4860:4802:36::a
ns4.google.com.         51471   IN      AAAA    2001:4860:4802:38::a

;; Query time: 7 msec
;; SERVER: 10.255.255.254#53(10.255.255.254) (UDP)
;; WHEN: Fri May 22 18:46:48 PKT 2026
;; MSG SIZE  rcvd: 303

```

Both records resolve the same hostname. The type determines which
address family is returned.

---

## Dual-Stack Behavior

A dual-stack host has both IPv4 and IPv6 addresses and can connect to
servers using either. The choice of protocol depends on what both sides
support and the system's preference settings.

```bash
curl -v https://google.com 2>&1 | grep -E "Trying|Connected|IPv"
```

```
* IPv6: 2a00:1450:4018:813::200e
* IPv4: 142.251.38.14
*   Trying 142.251.38.14:443...
* Connected to google.com (142.251.38.14) port 443
```

The `Trying` line shows which address family curl chose. On most
modern systems with IPv6 available, curl prefers IPv6 when the
destination has a AAAA record.

The address family can be forced explicitly:

```bash
# Force IPv4
curl -4 -s -o /dev/null -w "IPv4 status: %{http_code}\n" https://google.com

# Force IPv6
curl -6 -s -o /dev/null -w "IPv6 status: %{http_code}\n" https://google.com
```

```bash
# ipv4 output:
IPv4 status: 301

# ipv6 output:
IPv6 status: 000
```

If IPv6 is not available in the environment, the `-6` command will fail.
This is expected in some WSL2 configurations where the Windows host
network does not provide global IPv6 connectivity.

---

## Where IPv6 Appears in Practice

**Kubernetes:** Pod IPs can be IPv4, IPv6, or both depending on the
CNI plugin and cluster configuration. `kubectl get pods -o wide` may
show IPv6 pod addresses. CoreDNS serves both A and AAAA records for
services.

**AWS:** VPCs support dual-stack. EC2 instances can receive both IPv4
and IPv6 addresses. Security groups apply to both address families.
Load balancers can be configured as dualstack.

**Docker:** Docker supports IPv6 networking. When enabled, containers
receive both IPv4 and IPv6 addresses within the bridge network.

**Captures:** tcpdump captures IPv6 traffic automatically.

---

## Comparing IPv4 and IPv6 Side by Side

| Property | IPv4 | IPv6 |
|---|---|---|
| Address size | 32 bits | 128 bits |
| Notation | Decimal, dots | Hex, colons |
| Loopback | 127.0.0.1 | ::1 |
| Link-local range | 169.254.0.0/16 | fe80::/10 |
| DNS record type | A | AAAA |
| Subnet notation | /24, /20, /16 | /64 (standard) |
| ARP equivalent | ARP | NDP (Neighbour Discovery) |
| Private ranges | RFC 1918 | fc00::/7 (ULA) |

The important observation: the concepts are the same. Addressing,
subnets, routing, DNS resolution — IPv6 implements all of them.
The notation is different, the scale is different, but the mental
model from Lab 06 applies directly.

---

*Next: Lab 08 — Routing and Gateways*
*How the kernel makes routing decisions, how traceroute maps the*
*path a packet takes, and what happens at each hop between source*
*and destination.*