# Linux Networking — Module Học Tập

> Module gồm 4 phần: Lý thuyết, Lệnh thực chiến, Tình huống thực tế, và Kiểm tra.

---

## ① Lý thuyết

### Networking hoạt động như thế nào?

Trong Linux, networking là cách máy chủ của bạn **giao tiếp** với thế giới bên ngoài — từ request HTTP đến kết nối SSH giữa các server trong cluster K8s.

> **Lưu ý:** Mỗi gói tin di chuyển theo mô hình OSI: Application → Transport (TCP/UDP) → Network (IP) → Link (MAC). Khi debug networking, bạn phải biết đang xét tầng nào.

Ba khái niệm cốt lõi cần nắm vững:

- **Interface** — card mạng vật lý hoặc virtual (`eth0`, `ens33`, `lo`)
- **IP address** — địa chỉ của máy trong mạng
- **Port** — cửa ra vào của từng ứng dụng (`80`=HTTP, `22`=SSH, `443`=HTTPS)

---

### Lệnh `ip` — công cụ chính

`ip` là lệnh hiện đại thay thế `ifconfig` (deprecated). Nó quản lý interfaces, routes, và địa chỉ IP.

```bash
# Xem tất cả network interfaces + IP
ip addr show
ip a          # viết tắt

# Xem routing table
ip route show
ip r          # viết tắt

# Xem trạng thái interface cụ thể
ip addr show eth0

# Bật/tắt interface
ip link set eth0 up
ip link set eth0 down
```

> ⚠️ Lệnh `ifconfig` vẫn còn nhưng không có mặc định trên Ubuntu mới. Trong môi trường DevOps/K8s, luôn dùng `ip`.

---

### Lệnh `netstat` & `ss` — xem kết nối

`ss` (socket statistics) là phiên bản mới của `netstat`, nhanh hơn và đầy đủ hơn.

```bash
# Xem tất cả ports đang lắng nghe
ss -tulnp

# Giải thích flags:
# -t = TCP   -u = UDP   -l = listening
# -n = numeric (hiện port số, ko resolve DNS)
# -p = process (hiện PID và tên app)

# Tìm xem port 80 đang dùng bởi gì
ss -tulnp | grep :80

# Xem established connections
ss -tn
```

> 💡 Trong K8s, khi một pod không kết nối được service, `ss -tulnp` trên node là bước debug đầu tiên để xác nhận port có đang bind không.

---

### Lệnh `curl` — test HTTP

`curl` là công cụ test API/HTTP không thể thiếu trong DevOps — dùng mọi lúc từ kiểm tra endpoint đến debug Ingress.

```bash
# GET cơ bản
curl https://api.example.com

# Xem cả response headers
curl -I https://example.com      # chỉ headers
curl -v https://example.com      # verbose full

# POST JSON
curl -X POST https://api.example.com/data \
     -H "Content-Type: application/json" \
     -d '{"key": "value"}'

# Test nội bộ (không cần DNS)
curl http://localhost:8080/health
```

---

### Lệnh `ssh` & `scp` — truy cập từ xa

SSH là cách bạn quản lý server từ xa — đây là kỹ năng nền tảng của DevOps/SysAdmin.

```bash
# Kết nối SSH cơ bản
ssh user@192.168.1.10

# Dùng private key
ssh -i ~/.ssh/id_rsa user@server.com

# SSH với port khác (không phải 22)
ssh -p 2222 user@server.com

# Sao chép file đến server
scp file.txt user@server:/home/user/

# Sao chép thư mục
scp -r ./mydir user@server:/opt/

# Copy từ server về máy local
scp user@server:/var/log/app.log ./
```

> 💡 Trong Ansible/Kubespray, SSH được dùng để kết nối và cấu hình từng node trong cluster. Hiểu SSH key-based auth là bắt buộc.

---

## ② Lệnh thực chiến

> Những lệnh bạn sẽ dùng hàng ngày trong môi trường DevOps.

### 🔴 `ip addr` / `ip route` — Bắt buộc

```bash
ip addr show              # Xem IP của tất cả interfaces
ip route show             # Xem routing table
ip -s link                # Xem thống kê gói tin
```

**Dùng khi:** server không kết nối được mạng, cần xác nhận IP đã được gán chưa.

---

### 🔴 `ss -tulnp` — Bắt buộc

```bash
ss -tulnp                 # Xem tất cả ports đang listen
ss -tulnp | grep nginx    # Nginx đang bind port nào?
ss -tn state established  # Kết nối đang active
```

**Dùng khi:** service không start được, port conflict, kiểm tra firewall.

---

### 🔴 `ping` / `traceroute` — Bắt buộc

```bash
ping -c 4 8.8.8.8         # Test kết nối cơ bản (4 gói)
ping google.com           # Test DNS + kết nối
traceroute 8.8.8.8        # Xem đường đi từng hop
mtr 8.8.8.8               # traceroute real-time (tốt hơn)
```

**Dùng khi:** debug không kết nối được server, xác định điểm đứt mạng.

---

### 🟡 `curl` — Thường dùng

```bash
curl -I https://example.com       # Chỉ lấy headers
curl -v http://localhost:8080     # Full verbose
curl -o /dev/null -s -w "%{http_code}" https://example.com
                                  # Chỉ lấy status code
```

**Dùng khi:** test API, kiểm tra Ingress K8s, debug HTTP response.

---

### 🟡 `nslookup` / `dig` — Thường dùng

```bash
nslookup google.com               # DNS lookup cơ bản
dig google.com                    # Thông tin DNS chi tiết
dig google.com MX                 # Xem mail records
dig @8.8.8.8 google.com          # Query DNS server cụ thể
```

**Dùng khi:** debug DNS trong K8s (CoreDNS), domain không resolve được.

---

### 🔵 `tcpdump` — Nâng cao

```bash
tcpdump -i eth0               # Capture gói trên interface
tcpdump -i eth0 port 80       # Chỉ HTTP traffic
tcpdump -i eth0 host 10.0.0.5 # Traffic đến/từ IP cụ thể
tcpdump -w capture.pcap       # Lưu ra file (xem bằng Wireshark)
```

> ⚠️ Cần quyền root. Đây là công cụ deep-dive — dùng khi các lệnh cơ bản không đủ để diagnose.

---

## ③ Tình huống thực tế

> Những vấn đề bạn sẽ gặp khi làm DevOps Intern — cùng workflow debug.

### 🔴 Tình huống 1: Nginx không start — "Address already in use"

Service báo lỗi bind port 80 nhưng không biết gì đang chiếm.

```bash
# Bước 1: Xem port 80 đang bị chiếm bởi gì
ss -tulnp | grep :80

# Output ví dụ:
# tcp LISTEN 0 511 0.0.0.0:80 ... users:(("apache2",pid=1234))

# Bước 2: Kill process hoặc đổi port của Nginx
kill 1234          # hoặc
systemctl stop apache2
```

---

### 🟡 Tình huống 2: Không SSH được vào server mới

Vừa provision VM xong nhưng SSH treo hoặc từ chối kết nối.

```bash
# Bước 1: Ping xem server reachable không
ping -c 3 192.168.1.100

# Bước 2: Kiểm tra port 22 có mở không
ss -tulnp | grep :22           # trên server (nếu có access)
curl -v telnet://192.168.1.100:22  # từ máy local

# Bước 3: Xem SSH service status
systemctl status sshd
```

---

### 🟢 Tình huống 3: Copy file config lên nhiều server

Cần deploy file `nginx.conf` lên 3 servers cùng lúc.

```bash
# Cách 1: SCP từng server (manual)
scp nginx.conf user@10.0.0.1:/etc/nginx/
scp nginx.conf user@10.0.0.2:/etc/nginx/
scp nginx.conf user@10.0.0.3:/etc/nginx/

# Cách 2: Dùng Ansible (DevOps-proper)
ansible webservers -m copy \
  -a "src=nginx.conf dest=/etc/nginx/nginx.conf" \
  --become
```

> 💡 Trong thực tế, bạn sẽ dùng Ansible cho việc này — đây là lý do tại sao học networking và Ansible cần đi song song.

---

## ④ Kiểm tra — Quiz

### Câu 1
**Lệnh nào dùng để xem tất cả ports đang listen trên Linux hiện đại?**

- A. `netstat -an`
- B. `ss -tulnp` ✅
- C. `ip addr show`
- D. `curl localhost`

> **Giải thích:** `ss -tulnp` là cách hiện đại: `-t` TCP, `-u` UDP, `-l` listening, `-n` numeric, `-p` process.

---

### Câu 2
**Bạn muốn copy thư mục `configs/` lên server 10.0.0.5 tại `/opt/`. Lệnh nào đúng?**

- A. `ssh -r configs/ root@10.0.0.5:/opt/`
- B. `scp configs/ root@10.0.0.5:/opt/`
- C. `scp -r configs/ root@10.0.0.5:/opt/` ✅
- D. `cp -r configs/ root@10.0.0.5:/opt/`

> **Giải thích:** `scp` cần flag `-r` để copy đệ quy (recursive) toàn bộ thư mục.

---

### Câu 3
**Lệnh nào giúp kiểm tra con đường gói tin đi qua từng hop để đến đích?**

- A. `ping`
- B. `ip route`
- C. `traceroute` ✅
- D. `ss -tn`

> **Giải thích:** `traceroute` (hoặc `mtr`) hiển thị từng router gói tin đi qua — rất hữu ích khi debug mạng bị đứt ở đâu.

---

### Câu 4
**Nginx báo lỗi "port 80 already in use". Bước đầu tiên bạn làm là?**

- A. Restart server
- B. Đổi Nginx sang port 8080
- C. `ss -tulnp | grep :80` để xem process nào đang chiếm ✅
- D. Xóa file `/etc/nginx/nginx.conf`

> **Giải thích:** Root cause analysis — luôn xác định nguyên nhân trước. `ss -tulnp | grep :80` cho bạn biết chính xác process nào đang chiếm port.

---

### Câu 5
**Câu nào mô tả đúng sự khác biệt giữa `ip` và `ifconfig`?**

- A. `ifconfig` nhanh hơn `ip` trên Ubuntu
- B. `ip` là công cụ hiện đại, `ifconfig` deprecated và không có mặc định trên Ubuntu mới ✅
- C. Hai lệnh hoàn toàn giống nhau
- D. `ip` chỉ dùng cho IPv6

> **Giải thích:** `ifconfig` từ `net-tools` package, deprecated. Trên Ubuntu 18.04+, `ip` (từ `iproute2`) là standard.
