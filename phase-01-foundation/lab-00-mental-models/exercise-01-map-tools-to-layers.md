# Exercise 01 — Map Tools to Layers

## What I Did
Ran common networking commands and mapped each one to the protocol layer and problem domain it helps diagnose.
1. ip addr
2. ip route
3. ss -tulnp
4. ping -c 3 8.8.8.8
5. curl -v http://example.com 2>&1 | head -60

## What I Saw (Real Output)

### 1. ip addr
```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet 10.255.255.254/32 brd 10.255.255.254 scope global lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:f6:d1:96 brd ff:ff:ff:ff:ff:ff
    inet 172.20.193.120/20 brd 172.20.207.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fef6:d196/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether b6:bf:55:08:dc:c9 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```

### 2. ip route
```bash
default via 172.20.192.1 dev eth0 proto kernel 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
172.20.192.0/20 dev eth0 proto kernel scope link src 172.20.193.120 
```

### 3. ss -tulnp

```bash
Netid              State               Recv-Q              Send-Q                            Local Address:Port                             Peer Address:Port              Process                                         
udp                UNCONN              0                   0                                    127.0.0.54:53                                    0.0.0.0:*                                                                 
udp                UNCONN              0                   0                                 127.0.0.53%lo:53                                    0.0.0.0:*                                                                 
udp                UNCONN              0                   0                                10.255.255.254:53                                    0.0.0.0:*                                                                 
udp                UNCONN              0                   0                                     127.0.0.1:323                                   0.0.0.0:*                                                                 
udp                UNCONN              0                   0                                         [::1]:323                                      [::]:*                                                                 
tcp                LISTEN              0                   4096                              127.0.0.53%lo:53                                    0.0.0.0:*                                                                 
tcp                LISTEN              0                   1000                             10.255.255.254:53                                    0.0.0.0:*                                                                 
tcp                LISTEN              0                   511                                   127.0.0.1:38047                                 0.0.0.0:*                  users:(("node",pid=104964,fd=22))              
tcp                LISTEN              0                   4096                                 127.0.0.54:53                                    0.0.0.0:*       
```

### 4. ping -c 3 8.8.8.8

```bash
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=55.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=80.1 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=81.0 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2065ms
rtt min/avg/max/mdev = 55.202/72.112/81.001/11.962 ms
```

### 5. curl -v http://example.com 2>&1 | head -60

```bash
 % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host example.com:80 was resolved.
* IPv6: 2606:4700:10::ac42:93f3, 2606:4700:10::6814:179a
* IPv4: 104.20.23.154, 172.66.147.243
*   Trying 104.20.23.154:80...
* Connected to example.com (104.20.23.154) port 80
> GET / HTTP/1.1
> Host: example.com
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Thu, 14 May 2026 09:11:44 GMT
< Content-Type: text/html
< Transfer-Encoding: chunked
< Connection: keep-alive
< Server: cloudflare
< Last-Modified: Thu, 14 May 2026 05:31:28 GMT
< Allow: GET, HEAD
< Accept-Ranges: bytes
< Age: 10224
< cf-cache-status: HIT
< CF-RAY: 9fb8c6f9cfc7a75a-KHI
< 
{ [528 bytes data]
100   528    0   528    0     0   1443      0 --:--:-- --:--:-- --:--:--  1450
* Connection #0 to host example.com left intact
<!doctype html><html lang="en"><head><title>Example Domain</title><meta name="viewport" content="width=device-width, initial-scale=1"><style>body{background:#eee;width:60vw;margin:15vh auto;font-family:system-ui,sans-serif}h1{font-size:1.5em}div{opacity:0.8}a:link,a:visited{color:#348}</style></head><body><div><h1>Example Domain</h1><p>This domain is for use in documentation examples without needing permission. Avoid use in operations.</p><p><a href="https://iana.org/domains/example">Learn more</a></p></div></body></html>
```

## Explanation

`ip addr` operates at Layer 2 and Layer 3. It shows your interface name, MAC address (link/ether line), IP address, prefix length (/20 etc), and interface state. It cannot see any traffic — it only shows configuration.

`ip route` operates at Layer 3. It shows how your kernel decides where to send packets. It cannot see traffic either — it only shows routing rules.

`ss -tulnp` operates at Layer 4. It shows TCP and UDP sockets — what is listening, what is connected, and which process owns each socket. It sees connections, not packets.

`ping` operates at Layer 3 using ICMP. It sends echo-request packets and waits for echo-reply. It tells you whether a host is reachable at layer 3. It cannot tell you if a specific port or service is working.

`curl` operates at Layer 7. It speaks HTTP. But to do that it first does DNS (Layer 7), then TCP connect (Layer 4), then TLS if HTTPS (Layer 7), then sends HTTP (Layer 7). The -v output shows you all of this happening in sequence.

## Key Insight

Different networking tools observe different parts of the protocol stack.

- ip addr and ip route — see configuration, not traffic
- tcpdump and tshark — see raw traffic at layer 2 and above
- ss — sees connections at layer 4
- ping — tests reachability at layer 3
- curl and dig — speak application protocols at layer 7
- No single tool sees everything

Networking tools observe different parts of the stack, so debugging requires choosing the correct tool for the layer where the problem exists.