# Lab 04 — Ethernet Frames and Packet Structure

## What This Lab Covers

Every IP packet, every DNS query, every HTTP request — all of it travels
inside an Ethernet frame before it crosses a network interface.

The frame is Layer 2. It is the physical envelope. IP is the letter
inside the envelope. TCP or UDP is the type of delivery service. HTTP
is the message itself. The frame is what the network adapter actually
sends and receives — the outer wrapper that everything else rides in.

Lab 03 captured frames without examining their structure. This lab
opens them up.

---

## Prerequisites

Lab 03 completed. tcpdump and Wireshark are operational. The
`captures/` directory contains at least one `.pcap` file.

---

## The Frame Structure

An Ethernet frame has three meaningful sections before the payload:

```
┌─────────────────────────────────────────────────────────────────┐
│  Ethernet Frame                                                   │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐             │
│  │ Dst MAC      │  │ Src MAC      │  │ EtherType  │  Payload... │
│  │ 6 bytes      │  │ 6 bytes      │  │ 2 bytes    │             │
│  └──────────────┘  └──────────────┘  └────────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

**Destination MAC** — where this frame is going on the local network.

**Source MAC** — where it came from.

**EtherType** — what protocol is in the payload. This two-byte field
tells the receiving adapter how to interpret the bytes that follow.

| EtherType | Protocol |
|---|---|
| `0x0800` | IPv4 |
| `0x0806` | ARP |
| `0x86DD` | IPv6 |

The payload is the IP packet, ARP message, or IPv6 packet — whatever
the EtherType specifies. Traditional Ethernet networks commonly use an MTU of 1500 bytes.

In this WSL2 environment, the interface MTU is slightly smaller
(1472 bytes) because of virtualization and encapsulation overhead.

---

## Capturing Frames with MAC Addresses Visible

By default, tcpdump shows IP addresses in the summary line. The `-e`
flag adds Layer 2 information — source and destination MAC addresses.

```bash
sudo tcpdump -i eth0 -e -c 15
```

Generate mixed traffic in a second terminal:

```bash
ping -c 5 8.8.8.8
```

```
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
17:54:56.010968 00:15:5d:2b:f8:58 (oui Unknown) > 01:00:5e:7f:ff:fa (oui Unknown), ethertype IPv4 (0x0800), length 212: DESKTOP-11LCHAP.mshome.net.59487 > 239.255.255.250.1900: UDP, length 170
17:54:57.022464 00:15:5d:2b:f8:58 (oui Unknown) > 01:00:5e:7f:ff:fa (oui Unknown), ethertype IPv4 (0x0800), length 212: DESKTOP-11LCHAP.mshome.net.59487 > 239.255.255.250.1900: UDP, length 170
17:54:58.028016 00:15:5d:2b:f8:58 (oui Unknown) > 01:00:5e:7f:ff:fa (oui Unknown), ethertype IPv4 (0x0800), length 212: DESKTOP-11LCHAP.mshome.net.59487 > 239.255.255.250.1900: UDP, length 170
17:54:59.028211 00:15:5d:2b:f8:58 (oui Unknown) > 01:00:5e:7f:ff:fa (oui Unknown), ethertype IPv4 (0x0800), length 212: DESKTOP-11LCHAP.mshome.net.59487 > 239.255.255.250.1900: UDP, length 170
17:55:04.813764 00:15:5d:c3:7a:d2 (oui Unknown) > 00:15:5d:2b:f8:58 (oui Unknown), ethertype IPv4 (0x0800), length 98: 172.20.193.120 > dns.google: ICMP echo request, id 19299, seq 1, length 64
17:55:04.832985 00:15:5d:2b:f8:58 (oui Unknown) > 00:15:5d:c3:7a:d2 (oui Unknown), ethertype IPv4 (0x0800), length 98: dns.google > 172.20.193.120: ICMP echo reply, id 19299, seq 1, length 64
17:55:05.814317 00:15:5d:c3:7a:d2 (oui Unknown) > 00:15:5d:2b:f8:58 (oui Unknown), ethertype IPv4 (0x0800), length 98: 172.20.193.120 > dns.google: ICMP echo request, id 19299, seq 2, length 64
17:55:05.833825 00:15:5d:2b:f8:58 (oui Unknown) > 00:15:5d:c3:7a:d2 (oui Unknown), ethertype IPv4 (0x0800), length 98: dns.google > 172.20.193.120: ICMP echo reply, id 19299, seq 2, length 64
17:55:06.816122 00:15:5d:c3:7a:d2 (oui Unknown) > 00:15:5d:2b:f8:58 (oui Unknown), ethertype IPv4 (0x0800), length 98: 172.20.193.120 > dns.google: ICMP echo request, id 19299, seq 3, length 64
17:55:06.835931 00:15:5d:2b:f8:58 (oui Unknown) > 00:15:5d:c3:7a:d2 (oui Unknown), ethertype IPv4 (0x0800), length 98: dns.google > 172.20.193.120: ICMP echo reply, id 19299, seq 3, length 64
17:55:07.818187 00:15:5d:c3:7a:d2 (oui Unknown) > 00:15:5d:2b:f8:58 (oui Unknown), ethertype IPv4 (0x0800), length 98: 172.20.193.120 > dns.google: ICMP echo request, id 19299, seq 4, length 64
17:55:07.838723 00:15:5d:2b:f8:58 (oui Unknown) > 00:15:5d:c3:7a:d2 (oui Unknown), ethertype IPv4 (0x0800), length 98: dns.google > 172.20.193.120: ICMP echo reply, id 19299, seq 4, length 64
17:55:08.437566 00:15:5d:c3:7a:d2 (oui Unknown) > 00:15:5d:2b:f8:58 (oui Unknown), ethertype IPv4 (0x0800), length 90: 172.20.193.120.47793 > prod-ntp-3.ntp1.ps5.canonical.com.ntp: NTPv4, Client, length 48
17:55:08.556223 00:15:5d:2b:f8:58 (oui Unknown) > 00:15:5d:c3:7a:d2 (oui Unknown), ethertype IPv4 (0x0800), length 90: prod-ntp-3.ntp1.ps5.canonical.com.ntp > 172.20.193.120.47793: NTPv4, Server, length 48
17:55:09.668778 00:15:5d:c3:7a:d2 (oui Unknown) > 00:15:5d:2b:f8:58 (oui Unknown), ethertype IPv4 (0x0800), length 98: 172.20.193.120 > dns.google: ICMP echo request, id 19299, seq 5, length 64
15 packets captured
15 packets received by filter
0 packets dropped by kernel
```

The output now includes MAC addresses at the start of each line:

```
17:55:07.838723 00:15:5d:2b:f8:58 (oui Unknown) > 00:15:5d:c3:7a:d2 (oui Unknown), ethertype IPv4 (0x0800), ...
17:55:08.437566 00:15:5d:c3:7a:d2 (oui Unknown) > 00:15:5d:2b:f8:58 (oui Unknown), ethertype IPv4 (0x0800), ...
```

Two MAC addresses appear for every packet. The first is the source,
the second is the destination. Both are on the local segment — the
Windows host and the WSL2 instance communicating through the virtual
Ethernet interface.

This connection was already visible in Lab 03. The tshark output showed
`Microsoft_c3:7a:d2 → Microsoft_2b:f8:58` — those were the same MAC
addresses, displayed with vendor prefixes resolved. The `-e` flag in
tcpdump shows the same information in raw form.

The `(oui Unknown)` text means tcpdump could not map the MAC address
prefix to a known hardware vendor in its local vendor database.

---

## ARP — Layer 2 in Action

ARP (Address Resolution Protocol) operates at the Layer 2 / Layer 3
boundary. It solves a specific problem: before sending an IP packet to
another machine on the local network, the sender needs to know the
destination's MAC address. ARP finds it.

This was visible in Lab 03 — ARP packets appeared in the unfiltered
capture without any explanation. This lab examines them properly.

```bash
sudo tcpdump -i eth0 -e arp
```

In a second terminal, flush the ARP cache to force new ARP exchanges:

```bash
sudo ip neighbor flush all
ping -c 3 8.8.8.8
```

```
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
18:04:30.687781 00:15:5d:c3:7a:d2 (oui Unknown) > Broadcast, ethertype ARP (0x0806), length 42: Request who-has DESKTOP-11LCHAP.mshome.net tell 172.20.193.120, length 28
18:04:30.688005 00:15:5d:2b:f8:58 (oui Unknown) > 00:15:5d:c3:7a:d2 (oui Unknown), ethertype ARP (0x0806), length 42: Reply DESKTOP-11LCHAP.mshome.net is-at 00:15:5d:2b:f8:58 (oui Unknown), length 28
18:04:36.530996 00:15:5d:2b:f8:58 (oui Unknown) > 00:15:5d:c3:7a:d2 (oui Unknown), ethertype ARP (0x0806), length 42: Request who-has 172.20.193.120 (00:15:5d:c3:7a:d2 (oui Unknown)) tell DESKTOP-11LCHAP.mshome.net, length 28
18:04:36.531014 00:15:5d:c3:7a:d2 (oui Unknown) > 00:15:5d:2b:f8:58 (oui Unknown), ethertype ARP (0x0806), length 42: Reply 172.20.193.120 is-at 00:15:5d:c3:7a:d2 (oui Unknown), length 28
```

Two types of ARP messages appear:

**ARP Request** — broadcast to the entire local segment:
```
00:15:5d:c3:7a:d2 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806)
ARP, Request who-has 172.20.192.1 tell 172.20.193.120
```

The destination MAC is `ff:ff:ff:ff:ff:ff` — the Ethernet broadcast
address. A frame sent to this address is delivered to every device on
the local segment. The machine is asking: "whoever has IP 172.20.192.1,
reply with your MAC address."

**ARP Reply** — unicast response:
```
00:15:5d:2b:f8:58 > 00:15:5d:c3:7a:d2, ethertype ARP (0x0806)
ARP, Reply 172.20.192.1 is-at 00:15:5d:2b:f8:58
```

The Windows host responds directly to the WSL2 instance: "172.20.192.1
is at MAC 00:15:5d:2b:f8:58." The WSL2 instance stores this mapping in
its ARP table and uses it for all subsequent frames to that gateway.

Verify the ARP table was populated:

```bash
ip neighbor show
```

```
172.20.192.1 dev eth0 lladdr 00:15:5d:2b:f8:58 REACHABLE 
```

The gateway's MAC address now appears in the neighbour table. Every
IP packet sent toward the default gateway uses this MAC address as the
frame's destination until the entry expires.

---

## Reading Frame Structure in Wireshark

Open the `.pcap` file from Lab 03 in Wireshark, or start a new capture
on `vEthernet (WSL)` and generate traffic.

Click on any ICMP packet in the packet list. In the details panel,
expand **Ethernet II**:

```
▼ Ethernet II, Src: Microsoft_c3:7a:d2 (00:15:5d:c3:7a:d2),
                Dst: Microsoft_2b:f8:58 (00:15:5d:2b:f8:58)
    Destination: Microsoft_2b:f8:58 (00:15:5d:2b:f8:58)
    Source: Microsoft_c3:7a:d2 (00:15:5d:c3:7a:d2)
    Type: IPv4 (0x0800)
```

Three fields are visible: destination MAC, source MAC, EtherType.
The EtherType `0x0800` confirms the payload is an IPv4 packet.

Now click on an ARP packet and expand **Ethernet II** again:

```
▼ Ethernet II, Src: Microsoft_c3:7a:d2, Dst: Broadcast (ff:ff:ff:ff:ff:ff)
    Destination: Broadcast (ff:ff:ff:ff:ff:ff)
    Source: Microsoft_c3:7a:d2 (00:15:5d:c3:7a:d2)
    Type: ARP (0x0806)
```

The EtherType changes to `0x0806`. The destination is broadcast. This
is the same ARP request observed in tcpdump, now visible as individual
field values in the Wireshark frame.

Click on the **Type** field in the details panel. The corresponding
bytes highlight in the packet bytes panel at the bottom — two bytes,
`08 00` for IPv4 or `08 06` for ARP. This is the EtherType at the
byte level.

---

## Frame vs Packet

These two terms are often used interchangeably but they refer to
different layers:

| Term | Layer | Scope |
|---|---|---|
| Frame | Layer 2 | Local network only — MAC to MAC |
| Packet | Layer 3 | End-to-end across networks — IP to IP |

A frame only exists on a single network segment. When a packet crosses
a router, the router strips the incoming frame, reads the destination
IP, builds a new frame with new source and destination MAC addresses
for the next segment, and forwards it. The IP packet inside is
unchanged. The frame is replaced at every hop.

The IP packet carries the source and destination IPs of the original
sender and final receiver. The frame only needs to reach the next
device — the next hop. Its MAC addresses change at every router.

This distinction matters when reading captures from different points
in a network path. The same IP packet will show different MAC addresses
depending on where it was captured.

---

## MTU and Frame Size

```bash
ip addr show eth0 | grep mtu
```

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1472 qdisc mq state UP group default qlen 1000
```

The `mtu 1500` value seen in Lab 02 is the maximum Ethernet frame
payload size. A frame cannot carry more than 1500 bytes of IP data.
If an IP packet is larger, it gets fragmented before transmission —
split into multiple frames, each 1500 bytes or smaller, reassembled
at the destination.

MTU mismatches between network segments cause a specific and frustrating
failure: large packets are dropped silently, small packets succeed.
The connection appears to work for some traffic and hang for others.
This is a real production failure pattern covered in the debugging labs.

For now, the important point is that `mtu 1500` in `ip addr` and the
maximum frame payload size are the same value. The interface enforces it.

---

## Saving This Capture

```bash
sudo tcpdump -i eth0 -e -w captures/lab-04-ethernet-frames.pcap arp or icmp
```

Generate traffic:

```bash
sudo ip neigh flush all
ping -c 5 8.8.8.8
```

Stop with `Ctrl+C`. This capture contains both ARP and ICMP traffic —
the ARP exchange to find the gateway's MAC followed by the ICMP packets
that used it.

```bash
tshark -r captures/lab-04-ethernet-frames.pcap -e eth.src -e eth.dst \
  -e eth.type -T fields
```

```
00:15:5d:2b:f8:58       00:15:5d:c3:7a:d2       0x0806
00:15:5d:c3:7a:d2       00:15:5d:2b:f8:58       0x0806
00:15:5d:c3:7a:d2       00:15:5d:2b:f8:58       0x0806
00:15:5d:2b:f8:58       00:15:5d:c3:7a:d2       0x0806
00:15:5d:c3:7a:d2       ff:ff:ff:ff:ff:ff       0x0806
00:15:5d:2b:f8:58       00:15:5d:c3:7a:d2       0x0806
00:15:5d:c3:7a:d2       00:15:5d:2b:f8:58       0x0800
00:15:5d:2b:f8:58       00:15:5d:c3:7a:d2       0x0800
00:15:5d:c3:7a:d2       00:15:5d:2b:f8:58       0x0800
00:15:5d:2b:f8:58       00:15:5d:c3:7a:d2       0x0800
00:15:5d:c3:7a:d2       00:15:5d:2b:f8:58       0x0800
00:15:5d:2b:f8:58       00:15:5d:c3:7a:d2       0x0800
00:15:5d:c3:7a:d2       00:15:5d:2b:f8:58       0x0800
00:15:5d:2b:f8:58       00:15:5d:c3:7a:d2       0x0800
00:15:5d:c3:7a:d2       00:15:5d:2b:f8:58       0x0800
00:15:5d:2b:f8:58       00:15:5d:c3:7a:d2       0x0800
00:15:5d:2b:f8:58       00:15:5d:c3:7a:d2       0x0806
00:15:5d:c3:7a:d2       00:15:5d:2b:f8:58       0x0806
```

This extracts just the Ethernet fields from every packet: source MAC,
destination MAC, and EtherType. The pattern is clear — ARP requests
go to `ff:ff:ff:ff:ff:ff`, everything else goes to the gateway MAC.

---

## What This Established

Ethernet frames are the Layer 2 transport mechanism for local network communication. Every IP packet rides
inside one. The frame contains the MAC addresses of the local sender and
receiver, plus the EtherType that identifies the payload protocol.

ARP is how MAC addresses are discovered. On IPv4 Ethernet networks, ARP is normally how local MAC address
discovery happens before communication can begin. — the frame has nowhere to go.

The MTU limits how much data fits in a single frame. This limit
propagates up through every protocol layer above it.

The next lab examines ARP in more depth and introduces DHCP — the
protocol that assigns IP addresses before ARP or anything else can work.

---

*Next: Lab 05 — ARP and Local Networking*
*ARP is examined fully — request, reply, cache, and what happens when*
*it fails. DHCP is introduced as the protocol that makes ARP possible*
*by assigning IP addresses in the first place.*