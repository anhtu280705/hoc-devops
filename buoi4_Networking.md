# Networking cho DevOps/SRE — Tổng hợp kiến thức theo 5W1H

> **Mục tiêu:** Tài liệu này tổng hợp kiến thức Networking theo hướng **DevOps/SRE**, bám sát nội dung slide `Network Cơ Bản — OSI, TCP/IP, DNS` và mở rộng theo phạm vi kiến thức người học yêu cầu.
>
> **Khung tư duy xuyên suốt:** Với mỗi chủ đề, hãy trả lời 5 câu hỏi:
>
> - **What — Là gì?**
> - **Why — Tại sao cần?**
> - **Where — Hoạt động ở đâu?**
> - **When — Khi nào xảy ra / khi nào dùng?**
> - **Who — Thành phần nào tham gia?**
> - **How — Hoạt động như thế nào?**
>
> **Trọng tâm DevOps/SRE:** IP/CIDR/Subnetting → TCP/IP → ARP → DNS → Linux Networking → Firewall/TLS/SSH → HTTP → Proxy/Nginx → Load Balancer → Cloud Networking → Troubleshooting.
>
> **Nguồn:** Slide được cung cấp: Network Cơ Bản — OSI, TCP/IP, DNS; slide chính bao phủ Network Fundamentals, IP Addressing & Subnetting, Network Protocols, Network Services, Linux Networking, Network Security và Application Networking. fileciteturn0file0L37-L49

---

# 1. NETWORK FUNDAMENTALS

## 1.1. Networking là gì?

### What
Networking là tập hợp các **thiết bị, giao thức và cơ chế** giúp các máy tính/hệ thống trao đổi dữ liệu với nhau.

Ví dụ:

```text
Client
   │
   │ Request
   ▼
Network
   │
   ▼
Server
   │
   │ Response
   ▼
Client
```

### Why
DevOps/SRE cần Networking để:

- Triển khai ứng dụng.
- Kết nối server, container, Kubernetes.
- Hiểu traffic đi qua đâu.
- Phân tích lỗi kết nối.
- Cấu hình firewall.
- Thiết kế VPC/subnet/cloud.
- Hiểu HTTP, DNS, TLS, load balancing.

### Where
Networking xuất hiện ở:

```text
Laptop
Server
VM
Container
Kubernetes
Data Center
Cloud VPC
Internet
```

### When
Bất cứ khi nào một hệ thống cần giao tiếp:

```text
Application → Database
Client → API
Pod → Pod
VM → Internet
On-Premise → Cloud
```

### Who
Các thành phần thường gặp:

- NIC
- Switch
- Router
- Gateway
- Firewall
- Load Balancer
- Proxy
- DNS
- Server
- Client

### How
Dữ liệu được đóng gói qua các tầng mạng, truyền qua các thiết bị trung gian và được giải đóng gói tại phía nhận.

---

# 2. OSI MODEL VÀ TCP/IP MODEL

Slide phân biệt **OSI là mô hình lý thuyết/reference model**, còn **TCP/IP là mô hình thực tế được triển khai trong hệ thống mạng**. TCP/IP gộp 3 lớp trên cùng của OSI vào Application. fileciteturn0file0L56-L96

## 2.1. OSI 7 Layers

| Layer | Tên | Vai trò chính | Ví dụ |
|---|---|---|---|
| 7 | Application | Giao tiếp ứng dụng | HTTP, DNS, FTP |
| 6 | Presentation | Định dạng/mã hóa dữ liệu | Encoding, Encryption |
| 5 | Session | Quản lý phiên | Session |
| 4 | Transport | Giao tiếp end-to-end | TCP, UDP |
| 3 | Network | Định tuyến | IP, ICMP |
| 2 | Data Link | Frame/MAC | Ethernet, ARP |
| 1 | Physical | Truyền tín hiệu | Cáp, sóng |

Mnemonic:

```text
Application
Presentation
Session
Transport
Network
Data Link
Physical
```

## 2.2. TCP/IP Model

```text
Application
    │
    ├── HTTP
    ├── HTTPS
    ├── DNS
    └── FTP

Transport
    │
    ├── TCP
    └── UDP

Internet
    │
    ├── IP
    └── ICMP

Network Access
    │
    ├── Ethernet
    ├── Wi-Fi
    └── ARP
```

Slide nêu ví dụ HTTP TCP/80, HTTPS TCP/443, FTP TCP/21, DNS/53; TCP/UDP ở Transport; IP/ICMP ở Internet; Ethernet/Wi-Fi/ARP ở Network Access. fileciteturn0file0L76-L93

## 2.3. 5W1H

### What
OSI là mô hình tham chiếu 7 lớp. TCP/IP là stack thực tế.

### Why
Giúp chia nhỏ quá trình giao tiếp để:

- Học Networking.
- Thiết kế hệ thống.
- Debug lỗi.
- Xác định lỗi nằm ở tầng nào.

### Where
Trong toàn bộ quá trình:

```text
Application
↓
Transport
↓
Network
↓
Data Link
↓
Physical
```

### When
Khi troubleshoot:

```text
Không có link
→ Physical

Không có MAC/Frame
→ Data Link

Sai IP/Route
→ Network

TCP không connect
→ Transport

HTTP 500
→ Application
```

### Who
Mỗi tầng có giao thức/thành phần riêng.

### How
Dữ liệu đi xuống các tầng ở sender, được truyền qua network, rồi đi ngược lên các tầng ở receiver.

---

# 3. PDU, ENCAPSULATION VÀ DECAPSULATION

## 3.1. PDU là gì?

Tên dữ liệu thay đổi theo tầng:

```text
Application
    │
    │ Data
    ▼
Transport
    │
    │ Segment
    ▼
Network
    │
    │ Packet
    ▼
Data Link
    │
    │ Frame
    ▼
Physical
    │
    │ Bits
    ▼
Network
```

## 3.2. Encapsulation

### What
Encapsulation là quá trình mỗi tầng thêm header cần thiết vào dữ liệu.

### How

```text
Application Data
       │
       ▼
[TCP Header | Data]
       │
       ▼
[IP Header | TCP Header | Data]
       │
       ▼
[Ethernet Header | IP Header | TCP Header | Data | FCS]
       │
       ▼
Bits
```

## 3.3. Decapsulation

Ở máy nhận:

```text
Bits
 ↓
Frame
 ↓ remove Ethernet header
Packet
 ↓ remove IP header
Segment
 ↓ remove TCP header
Data
 ↓
Application
```

### Why
Để mỗi tầng chỉ xử lý phần thông tin thuộc trách nhiệm của mình.

---

# 4. NIC, MAC, ETHERNET, SWITCH, ROUTER

## 4.1. NIC

### What
NIC (Network Interface Card) là interface giúp thiết bị kết nối vào mạng.

### Where
Trên:

- Server
- Laptop
- VM
- Physical machine

Linux thường có interface:

```text
eth0
ens33
ens160
lo
docker0
```

Kiểm tra:

```bash
ip addr
ip link
```

---

## 4.2. MAC Address

### What
MAC là địa chỉ ở tầng Data Link, dùng để nhận diện interface trong mạng Layer 2.

### Where
Hoạt động chủ yếu trong cùng broadcast domain/LAN.

### When
Khi thiết bị gửi Ethernet Frame.

### How

```text
Source MAC
Destination MAC
        │
        ▼
Ethernet Frame
```

**Điểm rất quan trọng:**

- Cùng subnet → frame hướng đến MAC của host đích.
- Khác subnet → frame hướng đến MAC của default gateway.

---

## 4.3. Switch

### What
Switch hoạt động chủ yếu ở Layer 2 và forward frame dựa trên MAC address.

### Why
Kết nối các thiết bị trong cùng LAN/VLAN.

### Where
```text
PC ─┐
PC ─┼── Switch
Server ─┘
```

### Who
Switch sử dụng:

```text
MAC Address
MAC Table
Port
```

### How

```text
Host A
  │
  │ Frame: Dest MAC = Host B
  ▼
Switch
  │
  │ Tra MAC Table
  ▼
Port của Host B
```

---

## 4.4. Router

### What
Router kết nối các mạng khác nhau và route packet dựa trên IP.

### Why
Cho phép:

```text
Subnet A
   ↓
Router
   ↓
Subnet B
```

### Where
- Default Gateway.
- Kết nối LAN với WAN/Internet.
- Kết nối các subnet.

### How

```text
Client
  │
  │ Destination IP
  ▼
Router
  │
  │ Routing Table
  ▼
Next Hop
```

---

## 4.5. Switch vs Router

| Tiêu chí | Switch | Router |
|---|---|---|
| Layer chính | L2 | L3 |
| Địa chỉ chính | MAC | IP |
| Phạm vi | Cùng LAN/VLAN | Nhiều mạng |
| Bảng | MAC Table | Routing Table |
| Forward | Frame | Packet |
| Gateway | Không | Có thể là gateway |

Mnemonic:

```text
Switch = Same network
Router = Remote network
```

---

# 5. DATA FLOW: CLIENT → SWITCH → ROUTER → SERVER

## 5.1. Cùng subnet

```text
Client
IP: 192.168.1.10
       │
       │ ARP: Ai có 192.168.1.20?
       ▼
Switch
       │
       ▼
Server
IP: 192.168.1.20
```

### How

1. Client biết IP server.
2. Client kiểm tra server cùng subnet.
3. Client ARP để lấy MAC server.
4. Client tạo Ethernet Frame.
5. Switch tra MAC Table.
6. Switch forward frame đến server.
7. Server nhận frame và decapsulation.

---

## 5.2. Khác subnet

```text
Client
192.168.1.10
   │
   │ Destination = 10.0.0.10
   │
   │ ARP tìm MAC Gateway
   ▼
Switch
   │
   ▼
Default Gateway
192.168.1.1
   │
   │ Routing Table
   ▼
Router
   │
   ▼
Server
10.0.0.10
```

**Điểm cần nhớ:**

> Nếu server đích nằm khác subnet, client không ARP để tìm MAC của server đích. Client ARP để tìm MAC của Default Gateway.

---

# 6. IP ADDRESSING

Slide mô tả IPv4 gồm 32 bit, chia thành 4 octet, mỗi octet 8 bit. Slide cũng trình bày các dải private IP RFC 1918 và CIDR. fileciteturn0file0L108-L174

## 6.1. IPv4

### What

Ví dụ:

```text
192.168.10.5
```

Có:

```text
4 octet × 8 bit = 32 bit
```

### Why
Định danh logical address của host.

### Where
Layer 3.

---

## 6.2. Private IP

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

Dùng phổ biến trong:

- LAN.
- Data Center.
- VM.
- Docker.
- Cloud VPC.

---

# 7. CIDR VÀ SUBNETTING

## 7.1. CIDR là gì?

Ví dụ:

```text
192.168.10.35/27
```

`/27` nghĩa là:

```text
27 bit Network
5 bit Host
```

Tổng IP:

```text
2^(32 - 27) = 32
```

Usable host:

```text
32 - 2 = 30
```

Slide đưa bảng:

```text
/24 → 256 IP → 254 host
/25 → 128 IP → 126 host
/26 → 64 IP  → 62 host
/27 → 32 IP  → 30 host
/28 → 16 IP  → 14 host
/16 → 65536 IP → 65534 host
```

Công thức tổng quát:

```text
Total IP = 2^(32 - prefix)

Usable Host = Total IP - 2
```

fileciteturn0file0L147-L174

---

## 7.2. Phân tích 192.168.10.35/27

Block size:

```text
256 - 224 = 32
```

Các subnet:

```text
192.168.10.0
192.168.10.32
192.168.10.64
192.168.10.96
...
```

IP `192.168.10.35` thuộc:

```text
Network:
192.168.10.32/27

Broadcast:
192.168.10.63

First Host:
192.168.10.33

Last Host:
192.168.10.62

Usable Hosts:
30
```

---

## 7.3. 5W1H của Subnetting

### What
Chia một mạng lớn thành nhiều mạng nhỏ.

### Why
- Quản lý IP.
- Giảm broadcast domain.
- Phân tách hệ thống.
- Tăng khả năng kiểm soát security.
- Thiết kế cloud VPC.

### Where
```text
VPC
 ↓
Subnet
 ↓
Route Table
 ↓
Security
```

### When
Khi cần:

- Public/Private subnet.
- Tách App/DB.
- Phân vùng hệ thống.

### Who
- Network Engineer.
- DevOps.
- SRE.
- Cloud Engineer.

### How

```text
10.0.0.0/16
      │
      ├── Public Subnet
      ├── Private App Subnet
      └── Private DB Subnet
```

---

# 8. TCP

## 8.1. What

TCP là giao thức Transport Layer có:

- Connection-oriented.
- Reliable delivery.
- Ordered delivery.
- Flow control.
- Congestion control.

## 8.2. Why

Dùng khi dữ liệu cần độ tin cậy cao.

Ví dụ:

```text
HTTP/HTTPS
SSH
FTP
Database connections
```

## 8.3. Where

Layer 4.

## 8.4. Who

```text
Client TCP
     ↕
Server TCP
```

## 8.5. TCP 3-Way Handshake

```text
Client                         Server

  │
  │ SYN
  ├───────────────────────────►
  │
  │ SYN-ACK
  ◄───────────────────────────┤
  │
  │ ACK
  ├───────────────────────────►
  │
  ▼
ESTABLISHED
```

### How

1. Client gửi `SYN`.
2. Server trả `SYN-ACK`.
3. Client trả `ACK`.
4. Hai bên vào trạng thái `ESTABLISHED`.

---

## 8.6. TCP Termination

```text
Client                         Server

  │ FIN
  ├───────────────────────────►
  │
  │ ACK
  ◄───────────────────────────┤
  │
  │ FIN
  ◄───────────────────────────┤
  │
  │ ACK
  ├───────────────────────────►
  ▼
CLOSED
```

Các flag quan trọng:

```text
SYN
ACK
FIN
RST
```

---

## 8.7. TCP State

Cần hiểu:

```text
LISTEN
SYN-SENT
SYN-RECEIVED
ESTABLISHED
FIN-WAIT
TIME-WAIT
CLOSE-WAIT
```

Đặc biệt:

### TIME_WAIT
Thường xuất hiện phía chủ động đóng connection để đảm bảo các packet cũ hết hạn trước khi reuse.

### CLOSE_WAIT
Cho biết peer đã đóng kết nối nhưng local application chưa đóng socket.

---

# 9. UDP

## What
UDP là giao thức Transport Layer connectionless.

## Why
Đơn giản, overhead thấp, không cần handshake TCP.

## Where
Layer 4.

## When

Ví dụ:

```text
DNS
DHCP
VoIP
Streaming
```

## How

```text
Application
   │
   │ UDP Datagram
   ▼
Network
```

UDP không đảm bảo:

- Delivery.
- Order.
- Retransmission.

---

# 10. ICMP

## What
ICMP là giao thức dùng để truyền thông tin điều khiển và lỗi mạng.

## Why
Troubleshooting và kiểm tra reachability.

## Where
Internet/Network Layer.

## Flow: Ping

```text
Client
   │
   │ ICMP Echo Request
   ▼
Server
   │
   │ ICMP Echo Reply
   ▼
Client
```

Lệnh:

```bash
ping 8.8.8.8
```

---

## Traceroute

```text
Client
  │
  │ TTL = 1
  ▼
Router 1
  │
  │ TTL expired
  ▼
Client

TTL = 2
  ↓
Router 1
  ↓
Router 2
  ↓
Client

...
```

Traceroute giúp quan sát các hop trên đường đi.

---

# 11. ARP

## What
ARP ánh xạ:

```text
IPv4 Address
     ↓
MAC Address
```

## Why
Ethernet cần MAC để gửi Frame trong Layer 2.

## Where
Trong mạng IPv4 LAN.

## Flow

```text
Client
IP: 192.168.1.10
MAC: AA-AA
   │
   │ ARP Request:
   │ "Who has 192.168.1.20?"
   │ Broadcast
   ▼
LAN
   │
   ▼
Server
IP: 192.168.1.20
MAC: BB-BB
   │
   │ ARP Reply
   │ "192.168.1.20 is BB-BB"
   ▼
Client ARP Cache
```

Sau đó:

```text
IP
 ↓
ARP
 ↓
MAC
 ↓
Ethernet Frame
```

### Khác subnet

```text
Client
   │
   │ Destination IP = Server
   │
   │ ARP MAC của Gateway
   ▼
Default Gateway
   │
   │ Route IP packet
   ▼
Remote Server
```

---

# 12. DNS

Slide trình bày DNS Resolution Flow theo hướng Browser cache → OS Resolver/`/etc/hosts` → Recursive Resolver và các DNS record A, AAAA, CNAME, MX, TXT, NS. Slide cũng nêu UDP/53 cho query thông thường, TCP/53 cho zone transfer hoặc response lớn, và TTL quyết định thời gian cache. fileciteturn0file0L198-L276

## 12.1. What

DNS = Domain Name System.

Biến:

```text
google.com
```

thành:

```text
IP Address
```

## 12.2. Why

Con người nhớ domain dễ hơn IP.

Application cần IP để kết nối.

## 12.3. Where

```text
Application
   ↓
DNS Resolver
   ↓
DNS Infrastructure
   ↓
IP
```

## 12.4. DNS Resolution Flow

```text
Browser
   │
   │ 1. Check Browser DNS Cache
   ▼
OS Resolver
   │
   │ 2. Check /etc/hosts
   ▼
Recursive Resolver
   │
   │ 3. Nếu chưa có cache
   ▼
Root DNS
   │
   ▼
TLD DNS
   │
   ▼
Authoritative DNS
   │
   │ 4. Trả IP
   ▼
Recursive Resolver
   │
   │ 5. Cache theo TTL
   ▼
OS / Browser
   │
   ▼
Application
```

### 5W1H

**What:** Phân giải tên miền.

**Why:** Cho phép application dùng domain thay vì IP.

**Where:** Browser, OS, resolver, DNS hierarchy.

**When:** Trước khi application kết nối đến domain chưa có IP trong cache.

**Who:** Browser, OS resolver, recursive resolver, Root, TLD, Authoritative DNS.

**How:** Cache → resolver → Root → TLD → Authoritative → trả IP.

---

## 12.5. DNS Record

| Record | Ý nghĩa |
|---|---|
| A | Domain → IPv4 |
| AAAA | Domain → IPv6 |
| CNAME | Alias domain |
| MX | Mail server |
| NS | Nameserver |
| TXT | Text/SPF/DKIM/verification |

---

## 12.6. DNS Troubleshooting

Luôn phân tầng:

```text
Application
   ↓
DNS Resolve
   ↓
IP
   ↓
TCP Connect
   ↓
TLS
   ↓
HTTP
```

Ví dụ:

```text
DNS fail
→ Không có IP

DNS OK
→ TCP fail
→ Vấn đề route/firewall/port/service

TCP OK
→ TLS fail
→ Certificate/TLS

TLS OK
→ HTTP 5xx
→ Backend/Application
```

Lệnh:

```bash
dig google.com
nslookup google.com
resolvectl status
cat /etc/resolv.conf
```

---

# 13. DHCP

## What
DHCP tự động cấp network configuration.

## Cấp

```text
IP
Subnet Mask
Default Gateway
DNS
```

## DORA Flow

```text
Client
  │
  │ DHCP Discover
  ▼
DHCP Server
  │
  │ DHCP Offer
  ▼
Client
  │
  │ DHCP Request
  ▼
DHCP Server
  │
  │ DHCP ACK
  ▼
Client configured
```

## 5W1H

**What:** Cấp cấu hình IP tự động.

**Why:** Không cần cấu hình IP thủ công cho từng host.

**Where:** LAN, enterprise, VM network.

**When:** Khi host mới tham gia mạng hoặc cần lease mới.

**Who:** DHCP Client và DHCP Server.

**How:** Discover → Offer → Request → ACK.

---

# 14. NTP

## What
NTP đồng bộ thời gian giữa các máy.

## Why

Thời gian chính xác quan trọng cho:

- TLS Certificate.
- Log correlation.
- Distributed systems.
- Event ordering.

## Flow

```text
Client
   │
   │ NTP Request
   ▼
NTP Server
   │
   │ Time Response
   ▼
Client
```

---

# 15. LINUX NETWORKING

Đây là nhóm rất quan trọng với DevOps/SRE.

## 15.1. Interface

```bash
ip addr
ip link
```

Kiểm tra:

- IP.
- MAC.
- UP/DOWN.

Flow:

```text
Application
   ↓
Socket
   ↓
Network Interface
   ↓
NIC
   ↓
Network
```

---

## 15.2. Routing

```bash
ip route
```

Routing table quyết định packet đi đâu.

Ví dụ:

```text
Destination       Gateway       Interface
0.0.0.0/0         192.168.1.1   eth0
192.168.1.0/24    direct        eth0
```

Flow:

```text
Destination IP
      ↓
Routing Table
      ↓
Longest Prefix Match
      ↓
Next Hop
      ↓
Interface
```

---

## 15.3. ARP/Neighbor

```bash
ip neigh
```

Dùng để xem mapping:

```text
IP ↔ MAC
```

---

## 15.4. Port và Socket

### What

```text
IP
→ Host

Port
→ Service

Socket
→ Endpoint giao tiếp
```

Một TCP connection được xác định bởi:

```text
Source IP
Source Port
Destination IP
Destination Port
Protocol
```

Ví dụ:

```text
192.168.1.10:51532
        │
        │ TCP
        ▼
192.168.1.20:443
```

Kiểm tra:

```bash
ss -tulnp
```

---

## 15.5. Connectivity Troubleshooting

```bash
ping
traceroute
tracepath
mtr
```

Flow:

```text
ping
→ Reachability

traceroute
→ Path

mtr
→ Path + Packet Loss + Latency
```

---

## 15.6. HTTP Troubleshooting

```bash
curl -v https://example.com
```

Phân tích:

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP
```

---

## 15.7. Port Testing

```bash
nc -zv 192.168.1.10 443
```

Dùng để kiểm tra TCP port có connect được hay không.

---

## 15.8. Packet Capture

```bash
sudo tcpdump -i eth0 port 443
```

Có thể quan sát:

```text
Source IP
Source Port
Destination IP
Destination Port
TCP Flags
Packet Flow
```

Đây là công cụ cực kỳ quan trọng khi troubleshoot thực tế.

---

# 16. NETWORK SECURITY

## 16.1. Firewall

### What
Firewall kiểm soát traffic.

### Rules

```text
ALLOW
DENY
DROP
REJECT
```

### Inbound

```text
Internet
   │
   ▼
Firewall
   │
   ▼
Server
```

### Outbound

```text
Server
   │
   ▼
Firewall
   │
   ▼
Internet
```

### DROP vs REJECT

```text
DROP
→ Im lặng bỏ packet

REJECT
→ Từ chối và phản hồi
```

---

## 16.2. Stateful Firewall

Theo dõi trạng thái connection:

```text
Client → Server
       SYN

Server → Client
       SYN-ACK

Firewall biết connection
→ Cho phép traffic hợp lệ
```

---

## 16.3. Linux Firewall

Cần biết cơ bản:

```text
UFW
iptables
nftables
```

Kiểm tra port:

```bash
ss -tulnp
```

Best practice:

- Chỉ mở port cần thiết.
- Giới hạn IP SSH.
- Không mở SSH cho toàn Internet nếu không cần.

Slide cũng nhấn mạnh chỉ mở port cần thiết, giới hạn IP truy cập SSH và kiểm tra port listening bằng `ss`. fileciteturn0file0L376-L386

---

# 17. TLS / HTTPS

## What

HTTPS = HTTP chạy trên TLS.

```text
HTTP
 ↓
TLS
 ↓
HTTPS
```

## Why

TLS cung cấp:

- Encryption.
- Authentication.
- Integrity.

## Who

```text
Client
Server
Certificate
CA
```

## Flow TLS khái quát

```text
Client
  │
  │ ClientHello
  ▼
Server
  │
  │ ServerHello
  │ Certificate
  ▼
Client
  │
  │ Validate Certificate
  │ Key Exchange
  ▼
Server
  │
  │ Secure Session
  ▼
Encrypted HTTP
```

### Certificate

Certificate giúp client xác minh server.

```text
Server Certificate
       ↓
CA Trust
       ↓
Certificate Validation
       ↓
Trusted Server
```

### TLS Termination tại Nginx

```text
Client
  │ HTTPS
  ▼
Nginx
  │ TLS Termination
  │
  │ HTTP hoặc HTTPS
  ▼
Backend
```

---

# 18. SSH

## What

SSH là giao thức truy cập từ xa an toàn.

Mặc định:

```text
TCP 22
```

## Authentication

### Password

```text
Client
 ↓ Password
SSH Server
```

### SSH Key

```text
Client
 ├── Private Key
 └── Public Key
          │
          ▼
Server
~/.ssh/authorized_keys
```

Private key phải được giữ bí mật.

---

# 19. HTTP

Slide phân loại HTTP status theo nhóm 1xx, 2xx, 3xx, 4xx và 5xx; đây là nền tảng của Application Networking. fileciteturn0file0L399-L414

## 19.1. HTTP Request

```text
GET /users HTTP/1.1
Host: example.com
Content-Type: application/json

Body
```

Gồm:

- Method.
- URL.
- Path.
- Header.
- Body.

## Methods

```text
GET
POST
PUT
PATCH
DELETE
```

---

## 19.2. HTTP Response

```text
HTTP/1.1 200 OK

Headers

Body
```

## Status

```text
1xx → Informational
2xx → Success
3xx → Redirect
4xx → Client Error
5xx → Server Error
```

Quan trọng:

```text
200
301
302
400
401
403
404
500
502
503
504
```

---

# 20. HTTP REQUEST FLOW

```text
Client
   │
   │ DNS Resolve
   ▼
IP Address
   │
   │ TCP Handshake
   ▼
TCP Connection
   │
   │ TLS Handshake
   ▼
HTTPS Connection
   │
   │ HTTP Request
   ▼
Server / Proxy / Nginx
   │
   │ Application Processing
   ▼
HTTP Response
   │
   ▼
Client
```

Đây là một trong những flow quan trọng nhất đối với DevOps/SRE.

---

# 21. COOKIE VÀ SESSION

## Cookie

Cookie được lưu ở client và gửi lại theo request.

```text
Server
  │
  │ Set-Cookie
  ▼
Browser
  │
  │ Cookie
  ▼
Server
```

## Session

Server lưu trạng thái session.

```text
Client
  │ Cookie/Session ID
  ▼
Server
  │
  ▼
Session Store
```

---

# 22. FORWARD PROXY VS REVERSE PROXY

Slide phân biệt Forward Proxy đại diện cho Client và Reverse Proxy đại diện cho Server/Backend; Reverse Proxy thường phục vụ routing, load balancing, TLS termination và cache. fileciteturn0file0L423-L432

## Forward Proxy

```text
Client
   │
   ▼
Forward Proxy
   │
   ▼
Internet
```

### What
Proxy đại diện cho Client.

### Why
- Filtering.
- Cache.
- Ẩn danh.
- Corporate network control.

### Who
Client chủ động cấu hình.

---

## Reverse Proxy

```text
Client
   │
   ▼
Reverse Proxy
   │
   ├── Backend 1
   ├── Backend 2
   └── Backend 3
```

### What
Proxy đại diện cho Backend.

### Why
- Routing.
- TLS termination.
- Load balancing.
- Security.
- Cache.

### Who
Client thường không cần biết backend phía sau.

---

# 23. NGINX

## Nginx có thể làm gì?

```text
Nginx
 ├── Web Server
 ├── Static File Server
 ├── Reverse Proxy
 ├── Routing
 ├── TLS Termination
 ├── Load Balancer
 └── Health Check
```

## Static File

```text
Client
   │
   │ GET /index.html
   ▼
Nginx
   │
   │ Đọc file local
   ▼
/var/www/html/index.html
   │
   ▼
Client
```

Backend không cần xử lý request này.

## Reverse Proxy

```text
Client
   │ HTTPS
   ▼
Nginx
   │
   │ HTTP
   ▼
Backend
```

## Routing

Ví dụ:

```text
/api/*
   ↓
Backend API

/static/*
   ↓
Static File

/admin/*
   ↓
Admin Service
```

---

# 24. LOAD BALANCER

## What
Phân phối traffic đến nhiều backend.

## Why

- High Availability.
- Scalability.
- Failover.
- Traffic Distribution.

## Flow

```text
             ┌── App 1
             │
Client
   │         ├── App 2
   ▼         │
Load Balancer
             └── App 3
```

## Algorithms

### Round Robin

```text
Request 1 → App 1
Request 2 → App 2
Request 3 → App 3
Request 4 → App 1
```

### Least Connections

```text
Chọn backend
có ít connection nhất
```

## Health Check

```text
Load Balancer
   │
   ├── Health Check → App 1 → Healthy
   ├── Health Check → App 2 → Healthy
   └── Health Check → App 3 → Unhealthy

Traffic
   ├── App 1
   └── App 2
```

App 3 bị loại khỏi pool.

---

# 25. L4 VS L7 LOAD BALANCING

## L4

Dựa trên:

```text
IP
Port
TCP/UDP
```

```text
Client
   ↓
L4 LB
   ↓
Backend
```

Không cần hiểu sâu HTTP.

## L7

Hiểu:

```text
HTTP
Host
Path
Header
Cookie
```

Ví dụ:

```text
/api
   ↓
API Backend

/images
   ↓
Image Backend
```

---

# 26. CLOUD NETWORKING

## 26.1. VPC

### What
VPC là mạng logic riêng trong Cloud.

### Flow

```text
VPC
 ↓
CIDR
 ↓
Subnet
 ↓
Route Table
 ↓
Security
```

---

## 26.2. Public vs Private Subnet

```text
VPC
│
├── Public Subnet
│     └── Load Balancer
│
└── Private Subnet
      ├── Application
      └── Database
```

### Public Subnet
Có route ra Internet Gateway.

### Private Subnet
Không trực tiếp expose Internet.

---

# 27. INTERNET GATEWAY

```text
Public Subnet
     │
     ▼
Internet Gateway
     │
     ▼
Internet
```

Dùng cho resource public giao tiếp Internet.

---

# 28. NAT GATEWAY

```text
Private Subnet
     │
     ▼
NAT Gateway
     │
     ▼
Internet
```

Mục tiêu:

```text
Private Server
→ Có thể đi Internet outbound

Internet
→ Không trực tiếp initiate connection
→ Vào Private Server
```

---

# 29. SECURITY GROUP VS NACL

## Security Group

```text
Security Group
      ↓
Resource / ENI
```

Kiểm soát traffic ở cấp resource/network interface.

Đặc điểm thường gặp:

- Stateful.
- Rule theo port/protocol/source.

## NACL

```text
NACL
 ↓
Subnet
```

Kiểm soát traffic ở cấp subnet.

### Nhớ

```text
Security Group
→ Resource

NACL
→ Subnet
```

---

# 30. CLOUD LOAD BALANCER

Kiến trúc phổ biến:

```text
Internet
   │
   ▼
Load Balancer
   │
   ▼
Private Subnet
   │
   ▼
Application
   │
   ▼
Database
```

Mục tiêu:

- Không expose application trực tiếp.
- Tăng HA.
- Phân phối traffic.

---

# 31. VPN

```text
On-Premise
     │
     │ Encrypted Tunnel
     ▼
VPN
     │
     ▼
Cloud VPC
```

Dùng để kết nối mạng riêng giữa:

```text
Data Center
↔
Cloud
```

---

# 32. VPC PEERING

```text
VPC A
  │
  │ Peering
  │
VPC B
```

Cho phép các VPC giao tiếp qua private networking theo route được cấu hình.

---

# 33. NETWORK TROUBLESHOOTING & RCA

## 33.1. Luồng troubleshooting chuẩn

```text
1. Xác định triệu chứng
        ↓
2. Check Interface
        ↓
3. Check IP
        ↓
4. Check Routing
        ↓
5. Check ARP/Neighbor
        ↓
6. Check DNS
        ↓
7. Check Port/Socket
        ↓
8. Check Firewall
        ↓
9. Check TCP Connection
        ↓
10. Check TLS
        ↓
11. Check HTTP
        ↓
12. Packet Capture
        ↓
13. Xác định Root Cause
        ↓
14. Fix
        ↓
15. Verify
        ↓
16. RCA / Prevention
```

## 33.2. Bộ công cụ

| Mục tiêu | Công cụ |
|---|---|
| Interface | `ip addr`, `ip link` |
| Route | `ip route` |
| ARP | `ip neigh` |
| Port | `ss` |
| Process-Port | `lsof` |
| DNS | `dig`, `nslookup` |
| Reachability | `ping` |
| Path | `traceroute`, `tracepath` |
| Path + loss | `mtr` |
| HTTP | `curl`, `wget` |
| TCP port | `nc` |
| Packet | `tcpdump` |

## 33.3. Troubleshooting theo tầng

```text
Application
   │
   │ HTTP lỗi?
   ▼
TLS
   │
   │ Handshake lỗi?
   ▼
TCP
   │
   │ Connection?
   ▼
Port / Firewall
   │
   ▼
Routing
   │
   ▼
IP
   │
   ▼
Interface
```

---

# 34. LAB CHECKLIST

Slide lab đề xuất kiểm tra IP/SSH giữa 2 máy, dùng ping/traceroute/nslookup/dig, kiểm tra DNS bằng `/etc/resolv.conf`, `resolvectl` và `dig @8.8.8.8`. fileciteturn0file0L339-L366

## Lab 1: IP và SSH

```bash
ip addr show
ip route show
ping 192.168.x.x
ssh anhtu@192.168.x.x
```

## Lab 2: Network Tools

```bash
ping -c 4 google.com
traceroute google.com
nslookup google.com
dig google.com A
```

## Lab 3: DNS

```bash
cat /etc/resolv.conf
resolvectl status
dig @8.8.8.8 google.com
```

---

# 35. BẢNG 5W1H TỔNG HỢP

| Thành phần | What | Why | Where | When | Who | How |
|---|---|---|---|---|---|---|
| IP | Địa chỉ logic | Định danh host | L3 | Khi route | Host/Router | Routing |
| MAC | Địa chỉ interface | Giao tiếp L2 | LAN | Gửi frame | NIC/Switch | MAC Table |
| Switch | Forward frame | Kết nối LAN | L2 | Cùng mạng | Host/Switch | MAC |
| Router | Route packet | Kết nối subnet | L3 | Khác mạng | Router | Routing Table |
| ARP | IP→MAC | Gửi Ethernet | LAN | Cần MAC | Host | Request/Reply |
| TCP | Reliable transport | Tin cậy | L4 | HTTP/SSH | Client/Server | Handshake |
| UDP | Datagram | Low overhead | L4 | DNS/DHCP | Client/Server | Datagram |
| ICMP | Control/error | Troubleshoot | L3 | Ping/Traceroute | Host/Router | Echo/TTL |
| DNS | Name→IP | Resolve domain | Application | Trước connect | Resolver | Query |
| DHCP | Cấp IP | Auto config | LAN | Host join mạng | Client/Server | DORA |
| Firewall | Filter traffic | Security | Host/Network | Mọi traffic | Firewall | Rules |
| TLS | Secure channel | Encryption/Auth | Transport/Application boundary | HTTPS | Client/Server/CA | Handshake |
| HTTP | App protocol | Web communication | Application | Request/Response | Client/Server | Methods |
| Proxy | Trung gian | Control/Routing | Network/App | Traffic qua proxy | Client/Server | Forward/Reverse |
| Nginx | Web/Proxy | Serve/route | Edge | HTTP traffic | Nginx | Config |
| LB | Phân phối traffic | HA/Scale | Edge | Nhiều backend | LB/Apps | Algorithm |
| VPC | Virtual network | Cloud isolation | Cloud | Deploy cloud | Cloud | CIDR/Subnet |
| NAT | Private→Public | Outbound Internet | Cloud/Router | Private host cần Internet | NAT | Address translation |

---

# 36. LỘ TRÌNH HỌC CHO DEVOPS/SRE

## Giai đoạn 1 — Nền tảng

```text
OSI
 ↓
TCP/IP
 ↓
PDU
 ↓
Encapsulation
 ↓
MAC
 ↓
IP
 ↓
Switch
 ↓
Router
```

## Giai đoạn 2 — IP

```text
IPv4
 ↓
Binary
 ↓
Subnet Mask
 ↓
CIDR
 ↓
Subnetting
 ↓
Routing
```

## Giai đoạn 3 — Protocol

```text
TCP
 ↓
UDP
 ↓
ICMP
 ↓
ARP
```

## Giai đoạn 4 — Services

```text
DNS
 ↓
DHCP
 ↓
NTP
```

## Giai đoạn 5 — Linux

```text
ip
 ↓
route
 ↓
neigh
 ↓
ss
 ↓
ping
 ↓
traceroute
 ↓
mtr
 ↓
dig
 ↓
curl
 ↓
nc
 ↓
tcpdump
```

## Giai đoạn 6 — Security

```text
Firewall
 ↓
TLS
 ↓
HTTPS
 ↓
SSH
```

## Giai đoạn 7 — Application Networking

```text
HTTP
 ↓
Proxy
 ↓
Reverse Proxy
 ↓
Nginx
 ↓
Load Balancer
```

## Giai đoạn 8 — Cloud

```text
VPC
 ↓
Subnet
 ↓
Route Table
 ↓
Internet Gateway
 ↓
NAT Gateway
 ↓
Security Group
 ↓
NACL
 ↓
VPN
 ↓
VPC Peering
```

---

# 37. THỨ TỰ ƯU TIÊN CHO DEVOPS/SRE

## 🔴 Ưu tiên rất cao

1. IP Address.
2. CIDR.
3. Subnetting.
4. Routing.
5. TCP.
6. UDP.
7. ICMP.
8. ARP.
9. DNS.
10. Linux Networking.
11. HTTP/HTTPS.
12. Reverse Proxy.
13. Nginx.
14. Load Balancer.
15. Troubleshooting.

## 🟠 Ưu tiên cao

16. Firewall.
17. TLS.
18. SSH.
19. VPC.
20. Subnet.
21. Route Table.
22. NAT.
23. Internet Gateway.
24. Security Group.

## 🟡 Học sau

25. VLAN.
26. Trunk.
27. STP nâng cao.
28. IPv6 nâng cao.
29. Routing Protocol chuyên sâu.

---

# 38. KIẾN TRÚC TỔNG QUÁT CẦN NHỚ

```text
                         INTERNET
                             │
                             ▼
                       DNS Resolution
                             │
                             ▼
                          Public IP
                             │
                             ▼
                    Load Balancer / Nginx
                             │
                    ┌────────┴────────┐
                    ▼                 ▼
                 App 1              App 2
                    │                 │
                    └────────┬────────┘
                             ▼
                         Database
```

Nếu đặt trong Cloud:

```text
Internet
   │
   ▼
Internet Gateway
   │
   ▼
Public Subnet
   │
   ▼
Load Balancer
   │
   ▼
Private App Subnet
   │
   ▼
Application
   │
   ▼
Private DB Subnet
   │
   ▼
Database
```

Nếu App cần Internet outbound:

```text
Private App
    │
    ▼
NAT Gateway
    │
    ▼
Internet Gateway
    │
    ▼
Internet
```

---

# 39. TƯ DUY CỐT LÕI CỦA DEVOPS/SRE

Khi một service không truy cập được, đừng đoán. Hãy đi theo flow:

```text
1. Có Interface không?
        ↓
2. Có IP không?
        ↓
3. Có Route không?
        ↓
4. Có ARP/Neighbor không?
        ↓
5. DNS có resolve không?
        ↓
6. Port có LISTEN không?
        ↓
7. Firewall có chặn không?
        ↓
8. TCP có handshake không?
        ↓
9. TLS có thành công không?
        ↓
10. HTTP trả status gì?
        ↓
11. Backend có healthy không?
        ↓
12. Packet capture xác nhận điều gì?
```

### Một câu nhớ toàn bộ Networking:

```text
NAME
 ↓
DNS
 ↓
IP
 ↓
ROUTE
 ↓
ARP/MAC
 ↓
TCP/UDP
 ↓
TLS
 ↓
HTTP
 ↓
APPLICATION
```

Đây là chuỗi tư duy quan trọng nhất để chuyển từ **học Networking** sang **troubleshooting hệ thống thực tế** trong DevOps/SRE.

---

# 40. CHECKLIST TỰ ĐÁNH GIÁ

Bạn có thể coi mình đã nắm nền tảng Networking cho DevOps/SRE khi có thể tự trả lời:

- [ ] OSI và TCP/IP khác nhau thế nào?
- [ ] Data → Segment → Packet → Frame là gì?
- [ ] MAC hoạt động ở đâu?
- [ ] IP hoạt động ở đâu?
- [ ] Khi nào cần Default Gateway?
- [ ] Switch và Router khác nhau thế nào?
- [ ] ARP hoạt động ra sao?
- [ ] Vì sao khác subnet phải gửi frame đến Gateway?
- [ ] `192.168.10.35/27` thuộc subnet nào?
- [ ] TCP 3-way handshake diễn ra thế nào?
- [ ] TIME_WAIT và CLOSE_WAIT khác nhau thế nào?
- [ ] UDP dùng khi nào?
- [ ] Ping và traceroute dùng ICMP/TTL như thế nào?
- [ ] DNS Resolve Flow diễn ra thế nào?
- [ ] DHCP DORA là gì?
- [ ] `ip addr`, `ip route`, `ip neigh` dùng để làm gì?
- [ ] `ss -tulnp` kiểm tra gì?
- [ ] `dig` khác `nslookup` thế nào?
- [ ] `curl -v` giúp debug gì?
- [ ] `tcpdump` nhìn được gì?
- [ ] Firewall DROP và REJECT khác nhau thế nào?
- [ ] TLS handshake có vai trò gì?
- [ ] SSH key authentication hoạt động thế nào?
- [ ] HTTP Request gồm những thành phần nào?
- [ ] 502, 503, 504 thường gợi ý vấn đề gì?
- [ ] Forward Proxy và Reverse Proxy khác nhau thế nào?
- [ ] Nginx làm Web Server và Reverse Proxy ra sao?
- [ ] L4 và L7 Load Balancer khác nhau thế nào?
- [ ] Health Check dùng để làm gì?
- [ ] Public và Private Subnet khác nhau thế nào?
- [ ] NAT Gateway khác Internet Gateway thế nào?
- [ ] Security Group và NACL khác nhau thế nào?
- [ ] Khi một service không truy cập được, bạn sẽ troubleshoot theo thứ tự nào?

---

# KẾT LUẬN

Đối với DevOps/SRE, Networking không nên được học rời rạc theo từng lệnh. Hãy luôn nối thành một flow:

```text
Application
    ↓
DNS
    ↓
IP
    ↓
Routing
    ↓
ARP / MAC
    ↓
Ethernet
    ↓
TCP / UDP
    ↓
TLS
    ↓
HTTP
    ↓
Proxy / Nginx
    ↓
Load Balancer
    ↓
Backend
    ↓
Database
```

Khi chuyển sang Cloud:

```text
CIDR
 ↓
VPC
 ↓
Subnet
 ↓
Route Table
 ↓
IGW / NAT
 ↓
Security Group / NACL
 ↓
Load Balancer
 ↓
Application
```

**Ưu tiên học sâu nhất:** `IP + CIDR/Subnetting + Routing + TCP/IP + ARP + DNS + Linux Networking + HTTP/HTTPS + Nginx/Reverse Proxy + Load Balancer + Troubleshooting`.

Đây là phần kiến thức có giá trị thực tế cao nhất đối với mục tiêu **DevOps/SRE**, trong khi VLAN/STP và Routing Protocol chuyên sâu có thể học sau khi nền tảng trên đã vững.
