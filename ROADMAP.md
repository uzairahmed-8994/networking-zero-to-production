# Networking Zero to Production — Roadmap

> Practical networking from zero to production-level diagnosis.
> Every concept introduced through a command, not a definition.
> Built for DevOps and Cloud Engineers.
> Environment: WSL2 Ubuntu + Docker + AWS EC2

---

## How This Repo Works

- One lab = one folder = one README
- Each lab has one clear purpose
- You paste real output, not example output
- You write real confusion, not summaries
- You commit after every lab
- Sequential only — do not skip

---

## Lab Format

Every lab README follows this structure:

```
What This Lab Does     — one paragraph, the real problem it solves
Prerequisites          — which lab comes before
The Analogy            — real world comparison before any commands
Exercises              — run, paste output, explain, key insight
After This Lab         — what you can now do
```

---

# PHASE 1 — Foundation

**Goal:** Build the mental model and get every tool working before
touching a single packet.

---

## Lab 00 — How the Internet Works

**Folder:** `phase-01-foundation/lab-00-how-the-internet-works`
**Time:** ~1 hour reading
**Type:** Concept only — no commands

**What it covers:**
- The letter analogy — encapsulation explained simply
- OSI model — 7 layers, what each does, protocols at each layer
- TCP/IP model — 4 layers, what actually runs on every device
- How OSI maps to TCP/IP
- The complete URL journey — DNS, TCP, TLS, HTTP step by step
- Encapsulation diagram — how data gets wrapped at each layer
- Packet vocabulary — frame, packet, segment, datagram
- Tools map to layers — which tool sees which layer

**No exercises. Read and understand the map before using any tools.**

---

## Lab 01 — Tools Setup and Verification

**Folder:** `phase-01-foundation/lab-01-tools-setup`
**Time:** ~1 hour
**Type:** Installation and verification

**What it covers:**
- Install all tools in one command
- Verify each tool responds correctly
- Install Wireshark on Windows, capture WSL2 traffic
- Verify tshark works inside WSL2

**Tools installed:**
tcpdump, tshark, nmap, netcat, socat, mtr, ipcalc, conntrack,
dnsutils, iperf3, iftop, nethogs, net-tools, traceroute, curl,
wget, openssl

---

## Lab 02 — Reading Your Network

**Folder:** `phase-01-foundation/lab-02-reading-your-network`
**Time:** ~2 hours
**Type:** Practical — reading current state

**What it covers:**
- `ip addr` — interfaces, MAC addresses, IP addresses, MTU, interface state
- `ip route` — routing table, default gateway, subnet routes
- `ip route get` — ask kernel which route it uses for a specific destination
- `ss -tulnp` — listening sockets, ports, owning processes
- Start a server, see it appear in ss, stop it, see it disappear

**After this lab:** You can read your machine's complete network state
and understand every field.

---

# PHASE 2 — Packets and Frames

**Goal:** See real traffic. Capture it, read it, understand the structure.
Learn what a packet actually looks like before studying what is inside it.

---

## Lab 03 — Packet Capture Basics

**Folder:** `phase-02-packets-and-frames/lab-03-packet-capture-basics`
**Time:** ~3 hours
**Type:** Practical — tcpdump and Wireshark

**What it covers:**
- tcpdump — capture live traffic, read summary lines
- Save captures to .pcap file
- Open .pcap in Wireshark — packet list, details, bytes panels
- Filter by protocol: icmp, tcp, udp
- Filter by host and port
- Expand packet layers in Wireshark details panel

**After this lab:** You can capture traffic, save it, and open it in
Wireshark without getting lost.

---

## Lab 04 — Ethernet Frames and Packet Structure

**Folder:** `phase-02-packets-and-frames/lab-04-ethernet-frames`
**Time:** ~2 hours
**Type:** Practical — reading frame structure

**What it covers:**
- MAC addresses in a captured frame
- EtherType field — 0x0800 IPv4, 0x0806 ARP, 0x86DD IPv6
- Frame vs packet — layer 2 wrapper vs layer 3 payload
- MTU in action — what happens with large packets
- Encapsulation visible in Wireshark bytes panel

**After this lab:** You can identify every layer of a captured packet
and explain what each part is for.

---

## Lab 05 — ARP and Local Networking

**Folder:** `phase-02-packets-and-frames/lab-05-arp-local-networking`
**Time:** ~2 hours
**Type:** Practical — ARP and DHCP

**What it covers:**
- How ARP works — who has this IP, I have it
- Clear ARP cache, watch ARP exchange happen
- Read ARP request and reply in Wireshark
- ARP table — ip neigh show, states explained
- DHCP briefly — how your machine gets its IP in the first place

**Analogy:** ARP is asking your neighbours "who lives at number 42?"
before you can knock on the door.

**After this lab:** You understand how machines on the same network
find each other before any IP communication can happen.

---

# PHASE 3 — IP Layer

**Goal:** Understand addressing, routing, and ICMP.
Every cloud networking concept (VPC, subnets, route tables) maps
directly to what you learn here.

---

## Lab 06 — IP Addressing and Subnetting

**Folder:** `phase-03-ip-layer/lab-06-ip-addressing-subnetting`
**Time:** ~3 hours
**Type:** Practical — IP addressing

**What it covers:**
- Read full interface config with ip addr
- Subnet calculation by hand — network, broadcast, first/last host
- ipcalc to verify calculations
- Private IP ranges — 10.x, 172.16-31.x, 192.168.x — why they exist
- CIDR notation — what /24, /16, /20 means in practice
- Add and remove a virtual IP from an interface

**After this lab:** You can read any IP address and prefix length and
immediately know the network range, broadcast, and whether two IPs
are on the same subnet.

---

## Lab 07 — IPv6 Basics

**Folder:** `phase-03-ip-layer/lab-07-ipv6-basics`
**Time:** ~1.5 hours
**Type:** Practical — brief IPv6

**What it covers:**
- Read IPv6 addresses — link-local vs global
- ping6 and IPv6 loopback
- dig AAAA records
- Dual-stack — how your machine chooses IPv4 vs IPv6
- Why you need to know it but not master it

**After this lab:** You are not blocked when you encounter IPv6 in
production, cloud, or Kubernetes.

---

## Lab 08 — Routing and Gateways

**Folder:** `phase-03-ip-layer/lab-08-routing-gateways`
**Time:** ~3 hours
**Type:** Practical — routing

**What it covers:**
- ip route show — full routing table, all columns
- ip route get — ask kernel which route it uses
- Add and delete static routes
- Default gateway — what happens if you remove it
- Longest prefix match — how kernel picks between overlapping routes
- traceroute — map each hop to a routing decision
- ICMP — ping deep dive, TTL, time exceeded, unreachable types

**After this lab:** You understand how packets decide where to go.
Every AWS route table and VPC gateway maps directly to what you
learned here.

---

# PHASE 4 — Transport Layer

**Goal:** Understand TCP and UDP completely.
Every production connectivity issue is a transport layer problem.
Refused, timeout, reset — you will know exactly what each means.

---

## Lab 09 — UDP and Connectionless Communication

**Folder:** `phase-04-transport/lab-09-udp`
**Time:** ~2 hours
**Type:** Practical

**What it covers:**
- UDP packet structure — 8-byte header, no handshake
- Send and receive UDP with netcat
- Capture DNS over UDP — see the query and response
- What happens when UDP is dropped — silence, no retry
- Why DNS, DHCP, and streaming use UDP

**After this lab:** You understand when and why UDP is used and what
failed UDP looks like compared to failed TCP.

---

## Lab 10 — TCP Handshake and Connection Lifecycle

**Folder:** `phase-04-transport/lab-10-tcp-handshake`
**Time:** ~4 hours
**Type:** Practical — the most important transport lab

**What it covers:**
- Capture a full TCP connection with tcpdump
- Find SYN, SYN-ACK, ACK in Wireshark
- Read sequence numbers and acknowledgement numbers
- ss -tan — watch connection states: LISTEN, SYN-SENT, ESTABLISHED
- FIN vs RST — graceful close vs abrupt close
- TIME_WAIT — why it exists and how long it lasts
- Half-open connection — SYN with no reply

**Analogy:** TCP is a phone call. Both sides confirm before speaking.
Both sides say goodbye before hanging up. RST is someone cutting the line.

**After this lab:** You can read any ss output and explain exactly
what state every connection is in and why.

---

## Lab 11 — TCP Failures and Diagnosis

**Folder:** `phase-04-transport/lab-11-tcp-failures`
**Time:** ~3 hours
**Type:** Practical — deliberate failures

**What it covers:**
- Connection refused — RST returned immediately, port closed
- Connection timeout — SYN sent, nothing comes back
- Retransmissions — capture them, see exponential backoff
- DROP vs REJECT with iptables — silent failure vs immediate failure
- TCP keepalive — how long-running connections detect dead peers

**After this lab:** You know the exact difference between refused,
timeout, and reset. You will never confuse them again.

---

## Lab 12 — Network Namespaces

**Folder:** `phase-04-transport/lab-12-network-namespaces`
**Time:** ~3 hours
**Type:** Practical — foundation for Docker networking

**What it covers:**
- Create an isolated network namespace
- Create a veth pair — virtual cable between namespaces
- Connect two namespaces through a bridge
- Capture traffic inside a namespace
- Reproduce what Docker does manually — namespace + veth + bridge + NAT

**After this lab:** Docker networking is not magic. You built it by hand.

---

# PHASE 5 — Application Layer

**Goal:** DNS, HTTP, TLS, SSH. The protocols your applications speak.
The four things you debug most often in production.

---

## Lab 13 — DNS Resolution

**Folder:** `phase-05-application/lab-13-dns`
**Time:** ~4 hours
**Type:** Practical — most important application lab

**What it covers:**
- dig — read every section of a DNS response
- Capture DNS with tcpdump, see UDP query and response
- dig +trace — full resolution from root to authoritative
- TTL and caching — watch TTL decrement
- Record types — A, AAAA, CNAME, MX, TXT, PTR
- Break DNS three ways — block port 53, wrong resolver, NXDOMAIN
- DNS inside Docker containers

**Analogy:** DNS is your phone's contacts app. google.com is the name,
142.250.x.x is the number. Without DNS you would need to remember
every IP address.

**After this lab:** DNS failures are never a black box. You trace
them from your machine to the authoritative server and back.

---

## Lab 14 — HTTP Deep Dive

**Folder:** `phase-05-application/lab-14-http`
**Time:** ~3 hours
**Type:** Practical

**What it covers:**
- HTTP by hand with netcat — type a GET request, read raw response
- tcpdump -A to read HTTP headers in terminal
- HTTP methods — GET, POST, HEAD, OPTIONS
- Status codes — trigger 200, 301, 404, 500 deliberately
- Keep-alive connection reuse
- Follow TCP Stream in Wireshark — read full conversation as text
- HTTP/2 basics — ALPN negotiation
- CDN brief — how CloudFront changes the path your packets take

**After this lab:** You can debug HTTP with nothing but netcat and
tcpdump. No browser needed.

---

## Lab 15 — TLS and Certificates

**Folder:** `phase-05-application/lab-15-tls`
**Time:** ~4 hours
**Type:** Practical

**What it covers:**
- openssl s_client — connect and read full TLS handshake output
- Certificate chain — leaf, intermediate, root CA
- Wireshark TLS handshake — ClientHello, ServerHello, Certificate
- Create a self-signed certificate from scratch
- Certificate errors — expired, wrong hostname, untrusted CA
- SNI — how one IP serves multiple HTTPS domains
- mTLS — server and client both authenticate
- TLS termination at reverse proxy
- ALPN negotiation — how HTTP/2 is negotiated inside TLS

**After this lab:** Certificate errors are diagnosable from the
command line. TLS is not a black box.

---

## Lab 16 — SSH and VPN Tunneling

**Folder:** `phase-05-application/lab-16-ssh-vpn`
**Time:** ~3 hours
**Type:** Practical

**What it covers:**
- SSH as a protocol — not just a login tool
- SSH port forwarding — local and remote
- SSH SOCKS5 proxy — tunnel traffic through a remote host
- VPN concepts — what a site-to-site VPN is
- How to diagnose a VPN that is not working — the exact method
- ping and traceroute in VPN context — what they prove and what they do not

**After this lab:** You understand the VPN debugging scenario you
described — ping tests layer 3, telnet tests layer 4, and now you
know what to do when both fail.

---

# PHASE 6 — Traffic Management

**Goal:** Proxies and load balancing.
Every production application has a proxy in front of it.
Understanding this layer makes you able to diagnose 502s,
timeouts, and routing failures in production.

---

## Lab 17 — Proxies and Reverse Proxies

**Folder:** `phase-06-traffic-management/lab-17-proxies`
**Time:** ~3 hours
**Type:** Practical

**What it covers:**
- nginx as reverse proxy — capture traffic on both sides
- Forward proxy with tinyproxy
- SSH SOCKS5 tunnel as forward proxy
- X-Forwarded-For — original IP preserved through proxy
- See how proxy rewrites source IP in packet captures

**After this lab:** You can configure a reverse proxy, trace traffic
through it, and diagnose proxy-related failures.

---

## Lab 18 — Load Balancing

**Folder:** `phase-06-traffic-management/lab-18-load-balancing`
**Time:** ~3 hours
**Type:** Practical

**What it covers:**
- Round-robin with nginx upstreams
- Least-connections balancing
- Health checks — remove unhealthy backend automatically
- Sticky sessions with ip_hash
- Layer 4 vs Layer 7 load balancing — what each one sees
- Backend failure mid-request — capture the RST

**After this lab:** You understand what a load balancer does at the
packet level and can diagnose 502s from failed upstreams.

---

# PHASE 7 — Network Control

**Goal:** iptables, NAT, and traffic simulation.
Everything Docker and Kubernetes does with networking is iptables.
AWS Security Groups and NACLs are iptables with a cloud API.

---

## Lab 19 — NAT and iptables

**Folder:** `phase-07-network-control/lab-19-nat-iptables`
**Time:** ~5 hours
**Type:** Practical — the most important infrastructure lab

**What it covers:**
- Observe WSL2 NAT — your IP vs what the internet sees
- iptables NAT table — MASQUERADE, PREROUTING, POSTROUTING
- DNAT — port forwarding, redirect incoming traffic
- conntrack — connection tracking state table
- How Docker uses NAT for port publishing
- iptables filter table — INPUT, OUTPUT, FORWARD chains
- DROP vs REJECT — capture the difference
- Stateful rules with conntrack
- LOG target — see what is being dropped
- Save and restore rules

**After this lab:** You can read any iptables ruleset, write rules
from memory, and diagnose what is being blocked and why.

---

## Lab 20 — Traffic Simulation

**Folder:** `phase-07-network-control/lab-20-traffic-simulation`
**Time:** ~2 hours
**Type:** Practical — creating controlled failures

**What it covers:**
- tc netem — add artificial latency
- Packet loss simulation
- Packet corruption
- Bandwidth limiting
- Simulate a bad mobile or international connection
- See how TCP responds to loss vs delay

**After this lab:** You can reproduce any network condition locally
and observe exactly how applications and TCP behave under it.

---

# PHASE 8 — Debugging

**Goal:** Tie everything together into a repeatable method.
You have all the tools. This phase is how to use them together
systematically when something breaks.

---

## Lab 21 — Debugging Playbook

**Folder:** `phase-08-debugging/lab-21-debugging-playbook`
**Time:** ~3 hours
**Type:** Synthesis

**What it covers:**
- Layer-by-layer method — L1 to L7, stop at first failure
- "Can I reach the host?" — ping, mtr, traceroute
- "Can I reach the port?" — nc -zv, refused vs timeout
- "Is DNS resolving?" — dig @8.8.8.8, bypass local resolver
- "What is listening?" — ss -tulnp
- "What is being dropped?" — iptables LOG, conntrack
- Build your personal CHEATSHEET.md

**After this lab:** You have a method. Not just commands.
Any connectivity problem has a systematic diagnosis path.

---

## Lab 22 — Production Failure Scenarios

**Folder:** `phase-08-debugging/lab-22-production-failures`
**Time:** ~5 hours
**Type:** Diagnosis practice — broken environments

**The 8 scenarios:**
1. DNS server unreachable — block port 53, diagnose full failure chain
2. Wrong default gateway — nonexistent gateway, diagnose
3. iptables silently dropping SYN — service hangs, find the cause
4. TLS certificate expired — nginx with expired cert, diagnose
5. MTU mismatch — small MTU, large responses hang
6. Port conflict — two services same port, diagnose and fix
7. Container cannot resolve DNS — alpine nslookup fails
8. Reverse proxy 502 — nginx upstream to dead backend

**After this lab:** You have experienced and diagnosed 8 real
production failures. When they happen in production, you recognise them.

---

# PHASE 9 — Container Networking

**Goal:** Apply everything to Docker and Kubernetes.
Nothing is new here — it is the same Linux networking with
container abstractions on top.

---

## Lab 23 — Docker Networking

**Folder:** `phase-09-containers/lab-23-docker-networking`
**Time:** ~4 hours
**Type:** Practical

**What it covers:**
- docker0 bridge — ip link, brctl, tcpdump on the bridge
- Custom bridge networks — container DNS by name
- Host networking mode
- Container-to-container across networks
- Port publishing — find the iptables DNAT rule Docker creates
- docker-compose networking — DNS by service name
- docker network inspect

**After this lab:** Docker networking is the same as what you built
in Lab 12. You can diagnose any container connectivity issue.

---

## Lab 24 — Kubernetes Networking

**Folder:** `phase-09-containers/lab-24-kubernetes-networking`
**Time:** ~5 hours
**Type:** Practical — use minikube or kind

**What it covers:**
- Pod networking — every pod gets its own IP
- CNI plugin — how pods get IPs, what veth pairs are created
- ClusterIP service — find the iptables rules kube-proxy creates
- NodePort and LoadBalancer — external traffic path
- CoreDNS — service.namespace.svc.cluster.local resolution
- Debugging pod connectivity — apply the Lab 21 method inside K8s

**K8s networking = Linux networking you already know:**

| K8s concept | What it actually is |
|---|---|
| Pod IP | IP address in a network namespace |
| Pod-to-pod on same node | veth pair + bridge (Lab 12) |
| Service ClusterIP | iptables DNAT rule (Lab 19) |
| Service DNS | CoreDNS = DNS resolver (Lab 13) |
| Network Policy | iptables firewall rules (Lab 19) |
| Ingress | Reverse proxy (Lab 17) |
| CNI plugin | Automated namespace + veth + routing |

**After this lab:** K8s networking has no mystery. Every concept
maps to a lab you already completed.

---

# PHASE 10 — Cloud Networking

**Goal:** Apply everything to AWS.
AWS networking is Linux networking with a cloud API.
Every concept maps directly to something you built locally.

---

## Lab 25 — AWS Networking

**Folder:** `phase-10-cloud/lab-25-aws-networking`
**Time:** ~5 hours
**Type:** Practical — requires AWS account (free tier sufficient)

**What it covers:**
- VPC and subnets — map to Lab 12 namespaces
- Security groups — stateful like conntrack + iptables ESTABLISHED
- NACLs — stateless like iptables without conntrack
- Internet gateway — your default route in cloud
- VPC Flow Logs — your tcpdump equivalent in AWS
- Ephemeral ports — return traffic behavior
- Public/private subnet architecture
- NAT Gateway vs Internet Gateway
- VPC peering — routing between isolated networks
- Debug EC2 connectivity failure step by step

**Every AWS concept mapped:**

| AWS concept | Local equivalent |
|---|---|
| VPC | Network namespace (Lab 12) |
| Subnet | Bridge network |
| Route table | ip route (Lab 08) |
| Internet Gateway | Default gateway |
| NAT Gateway | iptables MASQUERADE (Lab 19) |
| Security Group | iptables stateful rules (Lab 19) |
| NACL | iptables stateless rules (Lab 19) |
| VPC Flow Logs | tcpdump (Lab 03) |

**After this lab:** AWS networking is familiar, not mysterious.
You can design VPCs, diagnose connectivity failures, and read
Flow Logs the same way you read tcpdump output.

---

# PHASE 11 — Modern Concepts (Optional)

**Goal:** Exposure to where networking is going.
Do not do this phase before completing everything else.

---

## Lab 26 — Modern Networking Concepts

**Folder:** `phase-11-modern-optional/lab-26-modern-concepts`
**Time:** ~2 hours
**Type:** Reading and light exploration

**What it covers:**
- QUIC and HTTP/3 — reliability moved from kernel to userspace
- eBPF — run programs in the kernel without modules
- Cilium — eBPF-based CNI replacing iptables in Kubernetes
- Service mesh — Envoy, mTLS between services, Istio/Linkerd concepts

---

# Summary

| Phase | Labs | Approx Time |
|---|---|---|
| 1 — Foundation | 00, 01, 02 | 4 hours |
| 2 — Packets and Frames | 03, 04, 05 | 7 hours |
| 3 — IP Layer | 06, 07, 08 | 7.5 hours |
| 4 — Transport | 09, 10, 11, 12 | 12 hours |
| 5 — Application | 13, 14, 15, 16 | 14 hours |
| 6 — Traffic Management | 17, 18 | 6 hours |
| 7 — Network Control | 19, 20 | 7 hours |
| 8 — Debugging | 21, 22 | 8 hours |
| 9 — Container Networking | 23, 24 | 9 hours |
| 10 — Cloud Networking | 25 | 5 hours |
| 11 — Modern (optional) | 26 | 2 hours |
| **Total** | **26 labs** | **~81 hours** |

At 1.5 hours per day on weekdays: approximately 10-11 weeks.

---

## Learning Checkpoints

**After Phase 4 — Transport:**
You can read any tcpdump output and explain what is happening.
You can diagnose TCP connection failures from symptoms alone.

**After Phase 7 — Network Control:**
You understand what Docker and Kubernetes do at the network level.
iptables has no mystery.

**After Phase 8 — Debugging:**
You have a method. Any connectivity failure has a systematic path.
You have diagnosed 8 real production scenarios.

**After Phase 10 — Cloud:**
AWS networking is Linux networking with labels.
You can design, build, and debug VPC architectures.
Kubernetes networking makes complete sense.

---

*Repo started: [date]*
*Environment: WSL2 Ubuntu 22.04 + Docker Engine + EC2 Amazon Linux*