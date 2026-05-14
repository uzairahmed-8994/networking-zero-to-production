# Tools Reference

Every tool used in this repo with its purpose, protocol layer, and most useful flags.
Use this as a quick lookup when you forget a flag or need to know which tool to reach for.

---

## Quick Decision Guide

| I want to... | Use this |
|---|---|
| Capture raw packets | `tcpdump` or `tshark` |
| Visually inspect a capture | `Wireshark` |
| See my interfaces and IP addresses | `ip addr` |
| See my routing table | `ip route` |
| Find where a packet is going | `ip route get <dst>` |
| See all open connections and listening ports | `ss -tulnp` |
| Check if a port is reachable | `nc -zv host port` |
| Make a raw TCP or UDP connection | `nc` |
| Test layer 3 reachability | `ping` |
| Trace the path to a destination | `traceroute` or `mtr` |
| See per-hop latency and loss in real time | `mtr` |
| Query DNS | `dig` |
| Make HTTP requests and see everything | `curl -v` |
| Inspect TLS certificates | `openssl s_client` |
| Read or write firewall rules | `iptables` |
| See NAT and connection tracking state | `conntrack -L` |
| Add network delay or packet loss | `tc netem` |
| Measure bandwidth between two hosts | `iperf3` |
| See live bandwidth per connection | `iftop` |
| See live bandwidth per process | `nethogs` |
| Calculate subnets | `ipcalc` |

---

## Tools Detail

### tcpdump
**Layer:** L2 and above
**Purpose:** Capture and display packets in the terminal. Your primary capture tool inside WSL2.

```bash
sudo tcpdump -i eth0                          # capture everything on eth0
sudo tcpdump -i eth0 -c 10                   # capture 10 packets then stop
sudo tcpdump -i eth0 icmp                    # filter by protocol
sudo tcpdump -i eth0 port 53                 # filter by port
sudo tcpdump -i eth0 host 8.8.8.8            # filter by host
sudo tcpdump -i eth0 -XX                     # show hex + ASCII dump
sudo tcpdump -i eth0 -w captures/file.pcap   # save to file
sudo tcpdump -r captures/file.pcap           # read saved file
sudo tcpdump -i eth0 -nn                     # no DNS resolution, no port names
sudo tcpdump -i eth0 'tcp port 80 and host 1.2.3.4'  # combined filter
```

---

### tshark
**Layer:** L2 and above
**Purpose:** Terminal version of Wireshark. Same engine, more filtering power.

```bash
sudo tshark -i eth0                          # capture like tcpdump
sudo tshark -i eth0 -c 10                   # capture 10 packets
sudo tshark -i eth0 -Y "icmp"               # display filter (same as Wireshark)
sudo tshark -i eth0 -Y "tcp.port == 80"     # filter by TCP port
sudo tshark -r file.pcap                    # read saved capture
sudo tshark -i eth0 -T fields -e ip.src -e ip.dst -e tcp.dstport  # extract specific fields
```

---

### Wireshark
**Layer:** L2 and above
**Purpose:** Visual packet inspection. Run on Windows, capture `vEthernet (WSL)` adapter.

Key features to use:
- **Filter bar** — type display filters: `icmp`, `tcp.port == 80`, `dns`, `tls`
- **Follow TCP Stream** — right-click any TCP packet → Follow → TCP Stream
- **Expert Info** — Analyze menu → Expert Information (shows retransmissions, resets)
- **Statistics → TCP Stream Graphs → Throughput** — visualize bandwidth over time
- **Packet details panel** — expand every layer, read every field

---

### ip
**Layer:** L2–L3
**Purpose:** Interface management, routing, ARP table. Replaces ifconfig and route.

```bash
ip addr                          # show all interfaces and IP addresses
ip addr show eth0                # show specific interface
ip addr add 10.0.0.1/24 dev eth0 # add an IP address
ip addr del 10.0.0.1/24 dev eth0 # remove an IP address
ip link show                     # show interface state (UP/DOWN)
ip link set eth0 up              # bring interface up
ip route show                    # show routing table
ip route get 8.8.8.8             # show which route is used for this destination
ip route add 10.10.0.0/24 via 192.168.1.1  # add static route
ip route del 10.10.0.0/24        # remove static route
ip neigh show                    # show ARP table
ip neigh flush all               # clear ARP cache
ip -6 addr show                  # show IPv6 addresses
ip -6 route show                 # show IPv6 routing table
ip netns add ns1                 # create network namespace
ip netns exec ns1 ip addr        # run command inside namespace
```

---

### ss
**Layer:** L4
**Purpose:** Show socket state. Replaces netstat. Use this to see what is listening and what is connected.

```bash
ss -tulnp                        # all listening TCP and UDP sockets with process names
ss -tan                          # all TCP connections with numeric addresses
ss -tp                           # TCP connections with process names
ss -tp state established         # only established connections
ss -tp state time-wait           # only TIME_WAIT connections
ss -tp state listen              # only listening sockets
ss -s                            # summary statistics
```

TCP states you will see: `LISTEN`, `SYN-SENT`, `SYN-RECV`, `ESTABLISHED`,
`FIN-WAIT-1`, `FIN-WAIT-2`, `TIME-WAIT`, `CLOSE-WAIT`, `CLOSED`

---

### nc (netcat)
**Layer:** L4
**Purpose:** Raw TCP and UDP client and server. The Swiss army knife for testing connectivity.

```bash
nc -l 9999                       # listen on TCP port 9999
nc localhost 9999                # connect to TCP port 9999
nc -u -l 9999                   # listen on UDP port 9999
nc -u localhost 9999             # connect via UDP
nc -zv google.com 443            # test if port is open (z=zero-io, v=verbose)
nc -zv host 20-80                # scan port range
nc -w 5 localhost 9999           # connect with 5 second timeout
echo "hello" | nc localhost 9999 # send data and close
```

---

### ping
**Layer:** L3 (uses ICMP)
**Purpose:** Test basic reachability. First tool to reach for when something is unreachable.

```bash
ping 8.8.8.8                     # ping continuously
ping -c 3 8.8.8.8                # send 3 packets only
ping -c 3 -i 0.2 8.8.8.8        # send 3 packets 0.2 seconds apart
ping -s 1400 8.8.8.8             # send large packet (test MTU)
ping -M do -s 1400 8.8.8.8      # do-not-fragment flag (MTU discovery)
ping -t 1 8.8.8.8                # TTL=1 (forces ICMP time exceeded from first hop)
ping6 ::1                        # ping IPv6 loopback
ping6 fe80::1%eth0               # ping IPv6 link-local (needs interface scope)
```

---

### traceroute
**Layer:** L3
**Purpose:** Show the path packets take to a destination by exploiting TTL.

```bash
traceroute 8.8.8.8               # trace path using UDP (default)
traceroute -I 8.8.8.8            # use ICMP instead of UDP
traceroute -T 8.8.8.8            # use TCP SYN (bypasses some firewalls)
traceroute -n 8.8.8.8            # no DNS resolution (faster)
traceroute -m 20 8.8.8.8         # maximum 20 hops
```

---

### mtr
**Layer:** L3
**Purpose:** Combines ping and traceroute. Shows per-hop latency and packet loss in real time. Better than traceroute for diagnosing where loss occurs.

```bash
mtr 8.8.8.8                      # interactive real-time view
mtr -n 8.8.8.8                   # no DNS resolution
mtr --report 8.8.8.8             # run and print a report (non-interactive)
mtr --report -c 20 8.8.8.8       # report with 20 cycles
```

---

### dig
**Layer:** L7 (DNS)
**Purpose:** Query DNS. More detailed output than nslookup. Use this for all DNS debugging.

```bash
dig google.com                   # query A record using default resolver
dig google.com A                 # explicitly query A record
dig google.com AAAA              # query IPv6 address
dig google.com MX                # query mail records
dig google.com TXT               # query text records
dig google.com CNAME             # query canonical name
dig -x 8.8.8.8                   # reverse DNS lookup (PTR record)
dig @8.8.8.8 google.com          # use specific DNS server (bypass local cache)
dig +trace google.com            # trace full resolution from root servers
dig +short google.com            # short output — just the answer
dig +nocmd +noall +answer google.com  # clean output — answer section only
```

---

### curl
**Layer:** L7
**Purpose:** Make HTTP and HTTPS requests. With -v flag shows the full conversation.

```bash
curl http://example.com                      # basic GET request
curl -v http://example.com                   # verbose — show headers and TLS
curl -I http://example.com                   # HEAD request — headers only
curl -X POST -d "key=val" http://example.com # POST with data
curl -H "Authorization: Bearer token" url    # custom header
curl --http2 -v https://google.com           # force HTTP/2
curl -4 https://google.com                   # force IPv4
curl -6 https://google.com                   # force IPv6
curl -k https://localhost                    # skip TLS certificate verification
curl --cert client.pem --key key.pem url     # use client certificate (mTLS)
curl --socks5 localhost:9090 http://example.com  # use SOCKS5 proxy
curl -w "\n%{time_total}s\n" http://example.com  # show total time
```

---

### openssl
**Layer:** L7 (TLS)
**Purpose:** TLS debugging and certificate creation. Use for any HTTPS or certificate issue.

```bash
openssl s_client -connect google.com:443          # connect and show full TLS handshake
openssl s_client -connect host:443 -servername hostname  # with SNI
openssl x509 -in cert.pem -text -noout            # read certificate fields
openssl x509 -in cert.pem -dates -noout           # show expiry dates only
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes  # create self-signed cert
openssl verify -CAfile ca.pem cert.pem            # verify cert against CA
openssl s_server -cert cert.pem -key key.pem -port 4433  # simple TLS server
```

---

### iptables
**Layer:** L3–L4
**Purpose:** Firewall rules, NAT, packet filtering. Runs on every Linux server, Docker host, and Kubernetes node.

```bash
iptables -L -v -n --line-numbers             # list all filter rules with counters
iptables -t nat -L -v -n                     # list NAT table rules
iptables -A INPUT -p tcp --dport 8080 -j DROP   # drop incoming TCP on port 8080
iptables -A INPUT -p tcp --dport 8080 -j REJECT # reject (sends RST back)
iptables -A OUTPUT -d 8.8.4.4 -j DROP          # drop outbound to specific IP
iptables -D INPUT 3                             # delete rule number 3
iptables -F                                     # flush all rules (careful)
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -j MASQUERADE  # NAT masquerade
iptables -t nat -A PREROUTING -p tcp --dport 9090 -j DNAT --to-destination 127.0.0.1:3000  # port forward
iptables -A INPUT -p tcp --dport 9999 -j LOG --log-prefix "BLOCKED: "  # log packets
iptables -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT  # stateful allow
iptables-save > rules.v4                        # save rules to file
iptables-restore < rules.v4                     # restore rules from file
```

---

### conntrack
**Layer:** L4
**Purpose:** View and manage the connection tracking table. Useful for NAT debugging.

```bash
conntrack -L                     # list all tracked connections
conntrack -L -p tcp              # only TCP connections
conntrack -C                     # count tracked connections
conntrack -E                     # watch connection events live
conntrack -F                     # flush the table (careful)
```

---

### tc (Traffic Control)
**Layer:** L2
**Purpose:** Shape, delay, and corrupt network traffic. Used to simulate bad network conditions.

```bash
sudo tc qdisc add dev eth0 root netem delay 100ms         # add 100ms latency
sudo tc qdisc add dev eth0 root netem loss 10%            # add 10% packet loss
sudo tc qdisc add dev eth0 root netem corrupt 5%          # corrupt 5% of packets
sudo tc qdisc add dev eth0 root netem delay 100ms 20ms    # 100ms +/- 20ms jitter
sudo tc qdisc add dev eth0 root netem delay 100ms loss 5% # combined
sudo tc qdisc add dev eth0 root tbf rate 1mbit burst 32kbit latency 400ms  # bandwidth limit
sudo tc qdisc show dev eth0      # show current rules
sudo tc qdisc del dev eth0 root  # remove all rules (always clean up after)
```

---

### iperf3
**Layer:** L4–L7
**Purpose:** Measure actual network bandwidth between two hosts.

```bash
iperf3 -s                        # start server (listen mode)
iperf3 -c localhost              # connect to server, run TCP bandwidth test
iperf3 -c localhost -u -b 1G    # UDP test at 1Gbps target
iperf3 -c localhost -t 30        # run for 30 seconds
iperf3 -c localhost -R           # reverse direction (server sends to client)
iperf3 -c localhost -P 4         # 4 parallel streams
```

---

### iftop
**Layer:** L3–L4
**Purpose:** Live bandwidth usage per connection. Like top but for network connections.

```bash
sudo iftop -i eth0               # monitor eth0
sudo iftop -i eth0 -n            # no DNS resolution (faster)
sudo iftop -i eth0 -P            # show port numbers
```

---

### nethogs
**Layer:** L7
**Purpose:** Live bandwidth usage per process. Answers "which process is using all the bandwidth?"

```bash
sudo nethogs eth0                # monitor eth0 by process
sudo nethogs -d 2 eth0           # update every 2 seconds
```

---

### ipcalc
**Layer:** L3
**Purpose:** Subnet calculation. Verify your manual subnet math.

```bash
ipcalc 192.168.1.0/24           # full breakdown of a subnet
ipcalc 192.168.1.100/24         # breakdown for a host address
ipcalc 10.0.0.0/8               # class A subnet
```

---

## WSL2 Notes

- Always use `sudo` with tcpdump, tshark, iftop, nethogs
- Primary interface is `eth0` — confirm with `ip addr`
- Wireshark runs on Windows, captures the `vEthernet (WSL)` adapter
- `tshark` inside WSL2 for terminal-based visual analysis
- `tc` requires `sudo modprobe sch_netem` if the netem module is not loaded
- iptables on Ubuntu 22+ uses nftables backend — if rules behave unexpectedly, also check `nft list ruleset`