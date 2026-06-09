# Bash Commands — System Monitoring

> Tổng hợp các lệnh dùng trong monitoring script: Disk, RAM, CPU, Log, Bash cơ bản.

---

## 1. Disk

### Xem dung lượng disk (human-readable)
```bash
df -h /
```
`df` = disk free. `-h` chuyển bytes → KB/MB/GB. `/` chỉ xem partition root.

### Lấy cột Use% (cột 5, dòng 2)
```bash
df -h / | awk 'NR==2 {print $5}'
```
`NR==2` = dòng thứ 2. `$5` = cột thứ 5.

### Xóa ký tự % để lấy số nguyên
```bash
df -h / | awk 'NR==2 {print $5}' | tr -d '%'
```
`tr -d '%'` xóa ký tự `%`. Kết quả: `"80%"` → `"80"`.

---

## 2. RAM

### Xem RAM usage theo MB
```bash
free -m
```
`free` hiển thị RAM và swap. `-m` = megabytes. `-h` = human-readable. `-g` = gigabytes.

### Lấy used (cột 3) và total (cột 2)
```bash
free -m | awk 'NR==2 {print $3, $2}'
```
`$3` = used, `$2` = total. In cả 2 số cách nhau bằng space.

### Tính phần trăm RAM (bash thuần)
```bash
RAM_USED=$(echo $READ | awk '{print $1}')
RAM_TOTAL=$(echo $READ | awk '{print $2}')
RAM_PCT=$(( RAM_USED * 100 / RAM_TOTAL ))
```
`$(( ))` = arithmetic expansion. Nhân 100 trước để tránh mất phần thập phân khi chia nguyên.

---

## 3. CPU

### File chứa load average real-time
```bash
cat /proc/loadavg
# Output: 0.08 0.12 0.09 1/773 7894
# Thứ tự: load 1min  5min  15min  running/total  lastPID
```
`/proc/loadavg` là virtual file, kernel sinh ra realtime.

### Lấy load average 1 phút
```bash
cat /proc/loadavg | awk '{print $1}'
```
`$1` = field đầu tiên = load 1 phút.

### So sánh số thập phân với bc
```bash
echo "0.08 > 2.0" | bc
# Trả về 1 (true) hoặc 0 (false)
```
Bash không so sánh được số thập phân natively. `bc` = basic calculator hỗ trợ float.

Dùng trong `if`:
```bash
if [ $(echo "$CPU > $THRESHOLD_CPU" | bc) -eq 1 ]; then
    echo "ALERT"
fi
```

---

## 4. Log

### Append vào file log (không xóa cũ)
```bash
echo "message" >> /var/log/file.log
```
`>>` = append. `>` = overwrite (xóa hết nội dung cũ). Monitoring log **luôn dùng** `>>`.

### Redirect cả stdout và stderr vào log
```bash
command >> /var/log/file.log 2>&1
```
`2>&1` = redirect stderr (fd2) vào chỗ stdout (fd1) đang trỏ. Cả 2 đều vào file log.

### Lấy timestamp hiện tại
```bash
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
```

| Format | Ý nghĩa |
|--------|---------|
| `%Y`   | Năm 4 chữ số |
| `%m`   | Tháng (01–12) |
| `%d`   | Ngày (01–31) |
| `%H`   | Giờ (00–23) |
| `%M`   | Phút (00–59) |
| `%S`   | Giây (00–59) |

---

## 5. Bash cơ bản

### Khai báo biến
```bash
THRESHOLD=80          # Không có khoảng trắng quanh =
LOG="/var/log/app.log"
RESULT=$(command)     # Command substitution
```
> ⚠️ `THRESHOLD = 80` sẽ báo lỗi — bash không cho khoảng trắng quanh `=`.

### So sánh số nguyên trong if
```bash
if [ "$A" -gt "$B" ]; then
    echo "A lớn hơn B"
fi
```

| Toán tử | Ý nghĩa |
|---------|---------|
| `-gt`   | greater than (>) |
| `-lt`   | less than (<) |
| `-eq`   | equal (==) |
| `-ne`   | not equal (!=) |
| `-ge`   | greater or equal (>=) |
| `-le`   | less or equal (<=) |

> Luôn đặt `"$VAR"` trong dấu nháy kép. Khoảng trắng sau `[` và trước `]` là bắt buộc.

### Shebang line
```bash
#!/bin/bash
```
Dòng đầu tiên bắt buộc của mọi bash script. Báo OS dùng interpreter nào để chạy file.

---

## Tổng hợp — Pattern cốt lõi của monitoring script

```
Lấy số thô  →  Extract bằng awk  →  So sánh threshold  →  Ghi log
```

```bash
#!/bin/bash

THRESHOLD=80
LOG="/var/log/monitor.log"
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')

VALUE=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')

if [ "$VALUE" -gt "$THRESHOLD" ]; then
    echo "[$TIMESTAMP] [ALERT] Disk: ${VALUE}%" >> "$LOG"
else
    echo "[$TIMESTAMP] [OK]    Disk: ${VALUE}%" >> "$LOG"
fi
```
