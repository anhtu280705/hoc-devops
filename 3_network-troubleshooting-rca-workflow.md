# Network Troubleshooting & RCA Workflow — 16 Steps

## Tổng quan

```
🚨 INCIDENT
    │
    ├──→ 00. RECENT CHANGE?  ← CHECK SONG SONG, KHÔNG CHẶN LUỒNG CHÍNH
    │     → Nếu correlation mạnh + impact lớn → MITIGATE / ROLLBACK NGAY
    │       (vẫn tiếp tục điều tra bên dưới để CONFIRM root cause)
    │
    ▼
🔵 NETWORK TROUBLESHOOTING (01–08)
    │
01. Interface & IP configured?
    ↓
02. Routing & Gateway reachable?
    ↓
03. External connectivity?
    ↓
04. DNS Resolution working?
    ↓
05. Service running & Port listening?
    ↓
06. Firewall / Packet Filtering?
    ↓
07. TCP Connection established?
    ↓
08. Packet-level evidence
    ↓
NETWORK CONNECTIVITY CONFIRMED?
    ├─ NO  → quay lại các bước liên quan để xử lý tiếp
    └─ YES ↓
🟢 APPLICATION & RCA (09–16)
    │
09. Application Health?
    ↓
10. Logs & Errors?
    ↓
11. Metrics & Resource Health?
    ↓
12. Dependency / Upstream OK?
    ↓
13. Correlate Timeline
    ↓
14. IDENTIFY ROOT CAUSE
    ↓
15. FIX & VERIFY
    ↓
16. PREVENT RECURRENCE
    ↓
✅ RESOLVED
```

---

## 00. Recent change? (chạy song song)

**Mục đích:** Kiểm tra thay đổi gần nhất có trùng thời điểm xảy ra lỗi không — đây là câu hỏi rẻ nhất, tín hiệu mạnh nhất trong RCA.

```bash
git log --since="2 hours ago" --oneline
kubectl rollout history deployment/<name> -n <namespace>
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
find /etc -newermt "1 hour ago" -type f 2>/dev/null
```

**Quyết định:**
- Correlation mạnh (gap thời gian ngắn, đúng service bị ảnh hưởng) → **Rollback/mitigate ngay**, sau đó vẫn tiếp tục 01–16 để confirm.
- Correlation yếu → bỏ qua, đi thẳng vào bước 01.

---

## 🔵 Network Troubleshooting (01–08)

### 01. Interface & IP configured?
**Mục đích:** Xác nhận interface tồn tại, có UP và có IP đúng.
```bash
ip -br addr
ip link
nmcli device status
```

### 02. Routing & Gateway reachable?
**Mục đích:** Xác nhận máy có route đúng và đến được gateway.
```bash
ip route
ip route get 8.8.8.8
ping -c 4 <gateway>
```
→ Nếu không ping được gateway → vấn đề nội bộ (local L2/L3).

### 03. External connectivity?
**Mục đích:** Xác nhận máy có kết nối ra Internet/mạng ngoài — dùng IP thẳng để bypass DNS issue.
```bash
ping -c 4 8.8.8.8
curl -I https://google.com
```

### 04. DNS Resolution working?
**Mục đích:** Xác nhận domain name phân giải thành IP đúng. Chạy sau bước 03 để phân biệt lỗi DNS vs lỗi routing.
```bash
dig google.com
resolvectl status
cat /etc/resolv.conf
```

### 05. Service running & Port listening?
**Mục đích:** Xác nhận service đang chạy và port đang LISTEN — kiểm tra CẢ bind address.
```bash
systemctl status <svc>
ss -lntp | grep :<port>
netstat -lntp | grep :<port>
```
- `0.0.0.0:PORT` = OK, accept từ remote
- `127.0.0.1:PORT` = chỉ local, remote luôn FAIL — lỗi phổ biến nhất bị bỏ sót

### 06. Firewall / Packet Filtering?
**Mục đích:** Kiểm tra firewall có chặn traffic hay không.
```bash
iptables -V
# Check backend trước (legacy hoặc nf_tables) để tránh đọc trùng 1 rule set

iptables -L -n -v
# hoặc
nft list ruleset

ufw status verbose
```

### 07. TCP Connection established?
**Mục đích:** Xác nhận client có thiết lập được kết nối TCP đến server không (kiểm chứng 2 chiều: client + server).
```bash
nc -vz <IP> <PORT>
ss -tan state established
```

### 08. Packet-level evidence
**Mục đích:** Bắt gói tin để thấy chuyện gì đang thực sự xảy ra trên mạng — dùng khi 01–07 đều "pass" nhưng vẫn lỗi.
```bash
tcpdump -ni eth0 host <CLIENT_IP> and tcp port <PORT>
```
→ Soi SYN/SYN-ACK/ACK, RST bất ngờ, retransmit (Wireshark nếu cần phân tích sâu).

---

### Gate: Network connectivity confirmed?

- **NO** → quay lại các bước 01–08 liên quan để xử lý tiếp, chưa chuyển sang Application.
- **YES** → đi tiếp vào Application & RCA.

---

## 🟢 Application & RCA (09–16)

### 09. Application Health?
**Mục đích:** Network + TCP đều OK, nhưng application có thực sự trả lời đúng không.
```bash
curl localhost:PORT/health
curl <IP>:PORT/health
# HTTP 200 = OK
```

### 10. Logs & Errors?
**Mục đích:** Tìm lỗi/exception được ghi nhận trong log.
```bash
journalctl -u <svc> --no-pager
docker logs <container>
kubectl logs <pod>
grep -i "error\|fail\|exception"
```

### 11. Metrics & Resource Health?
**Mục đích:** Kiểm tra server có đang quá tải tài nguyên không.
```bash
top -b -n 1
free -h
df -h
iostat -xz 1 3
uptime
```

### 12. Dependency / Upstream OK?
**Mục đích:** Kiểm tra các service phụ thuộc (DB, Redis, API ngoài, DNS) có đang hoạt động không.
```bash
nc -vz <db_ip> 5432
redis-cli -h <redis_ip> ping
curl -f https://api.example.com/health
```

### 13. Correlate Timeline
**Mục đích:** Đối chiếu thời gian giữa Change + Logs + Metrics + Packet để tìm mối liên hệ.
```
Thời điểm deploy/config change
Thời điểm bắt đầu lỗi trong logs
Thời điểm spike CPU/RAM/lỗi
Thời điểm bắt đầu packet drop/RST
```
(Không có lệnh cố định — đây là bước tổng hợp bằng chứng đã thu thập ở các bước trên.)

### 14. Identify Root Cause
**Mục đích:** Xác định nguyên nhân gốc dựa trên toàn bộ bằng chứng.
```
Phân tích bằng chứng
Loại trừ các giả thuyết khác
Kết luận nguyên nhân gốc
```

### 15. Fix & Verify
**Mục đích:** Áp dụng fix và xác nhận dịch vụ hoạt động ổn định.
```
Apply fix / rollback / restart
Test lại: curl / health check
Monitor: HTTP 200, latency, error rate, CPU/RAM
```

### 16. Prevent Recurrence
**Mục đích:** Ngăn sự cố tái diễn trong tương lai.
```
Monitoring & Alerting
CI/CD test & quality gate
Canary deployment + Auto-rollback
Runbook & Documentation
Postmortem & Learnings
```

---

## Ghi chú quan trọng

1. **Bước 00 chạy song song, không chặn luồng chính** — đây là tín hiệu rẻ và mạnh nhất, nên luôn kiểm tra đầu tiên dù không đứng đầu chuỗi tuần tự 1-16.
2. **Gate "Network connectivity confirmed?"** là điểm rẽ bắt buộc — không debug Application khi Network chưa chắc chắn OK, tránh lãng phí thời gian sai tầng.
3. **Nguyên tắc RCA:** luôn đi từ layer thấp lên cao (L1 → L7), dừng lại ở layer đầu tiên FAIL, đặt giả thuyết cụ thể, rồi mới verify bằng lệnh — không đoán mò.
