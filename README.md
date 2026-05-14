# Networking Zero to Production

A hands-on networking lab repo built for practical understanding — not theory.

Every concept is introduced through a command, not a definition.
Real packet captures. Real output. Deliberate failures. Documented confusion.

---

## Who This Is For

- DevOps and Cloud Engineers who copy-paste networking commands without understanding them
- Anyone whose networking knowledge is theoretical but not practical
- Anyone who wants to go from zero to diagnosing real production failures

---

## What Makes This Different

Most networking resources teach you the OSI model and call it done.

This repo teaches you to:
- Capture real packets and read them field by field
- Watch TCP connections open, transfer data, and close
- Break things deliberately and diagnose them systematically
- Map every AWS and Docker networking concept to Linux commands you ran yourself

By the end you will not just know what DNS is — you will have traced a query from your machine to a root server and back, blocked port 53 and diagnosed the failure, and watched Docker's internal resolver respond to a container.

---

## Environment

| Component | Details |
|-----------|---------|
| Local OS | WSL2 Ubuntu 22.04 on Windows |
| Packet capture (visual) | Wireshark on Windows — captures `vEthernet (WSL)` adapter |
| Packet capture (terminal) | tcpdump and tshark inside WSL2 |
| Containers | Docker Engine installed directly in WSL2 |
| Cloud | AWS EC2 Amazon Linux (Phase 11 onwards) |

---

## How This Repo Works

### Every lab is a folder

```
lab-XX-name/
├── README.md            ← goal, prerequisites, tools, exercise list
├── exercise-01-name.md  ← one file per exercise
├── exercise-02-name.md
├── captures/            ← saved .pcap files from tcpdump
└── screenshots/         ← Wireshark screenshots
```

### Every exercise follows the same structure

```
## What I Did
## What I Saw (Real Output)
## What Confused Me
## Explanation
## Breaking It
## Key Insight
```

No summaries. No paraphrasing. Real output only.

### Rule: Sequential Only

Do not skip labs. Each one builds muscle memory and mental models
that the next lab assumes you already have.

---

## Learning Path

| Phase | Focus | Labs |
|-------|-------|------|
| 01 | Foundation | Mental models, tool setup |
| 02 | Layer 2 | Packet capture, Ethernet frames, ARP, DHCP |
| 03 | IP Layer | ICMP, IP addressing, IPv6, routing |
| 04 | Transport | UDP, TCP handshake, TCP failures, namespaces |
| 05 | Application | DNS, HTTP, TLS, proxies |
| 06 | Load Balancing | nginx upstreams, health checks, L4 vs L7 |
| 07 | Network Control | NAT, iptables, traffic simulation |
| 08 | Performance | iperf3, mtr, throughput analysis |
| 09 | Debugging | Playbook, 12 production failure scenarios |
| 10 | Containers | Docker networking, Kubernetes networking |
| 11 | Cloud | AWS VPC, security groups, NACLs, Flow Logs |
| 12 | Modern (Optional) | QUIC, HTTP/3, eBPF, Cilium, service mesh |

---

## Checkpoints

**After Phase 04 — Transport:**
You can read any tcpdump output and explain what is happening at L2 through L4.

**After Phase 07 — Network Control:**
You can configure iptables, NAT, and traffic shaping.
You can replicate what Docker and Kubernetes do at the network level.

**After Phase 09 — Debugging:**
You have a repeatable diagnosis method and have lived through 12 production failures.
Networking is no longer your weakest link.

**After Phase 11 — Cloud:**
Every AWS networking concept maps to a Linux command you already ran yourself.
The cloud is familiar, not mysterious.

---

## Tools Used

See [TOOLS.md](./TOOLS.md) for the full reference.

Core tools used throughout: `tcpdump`, `tshark`, `Wireshark`, `ip`, `ss`,
`netcat`, `ping`, `traceroute`, `mtr`, `dig`, `curl`, `iptables`, `openssl`

---

## Related

Built using the same method as my Docker zero-to-production repo.
Same philosophy: real confusion → document → understand → move forward.