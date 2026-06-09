# System Resource Monitor Script

Script bash giám sát tài nguyên hệ thống (Disk, RAM, CPU), ghi log cảnh báo khi vượt ngưỡng threshold.

---

## Cấu hình mặc định

| Biến | Giá trị | Mô tả |
|------|---------|-------|
| `THRESHOLD_DISK` | `80` | Ngưỡng Disk usage (%) |
| `THRESHOLD_RAM` | `85` | Ngưỡng RAM usage (%) |
| `THRESHOLD_CPU` | `2.0` | Ngưỡng CPU load average (1 phút) |
| `LOG` | `/home/anhtu/health_check.log` | Đường dẫn file log |
| `TIMESTAMP` | `$(date '+%F %T')` | Định dạng: `YYYY-MM-DD HH:MM:SS` |

---

## Script đầy đủ

```bash
#!/bin/bash

THRESHOLD_DISK=80
THRESHOLD_RAM=85
THRESHOLD_CPU=2.0
LOG="/home/anhtu/health_check.log"
TIMESTAMP=$(date '+%F %T')

#------CHECK DISK------
DISK=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')
if [ "$DISK" -gt "$THRESHOLD_DISK" ]; then
        echo "[$TIMESTAMP] [ALERT] Disk: ${DISK}%" >> "$LOG"
else
        echo "[$TIMESTAMP] [OK] Disk: ${DISK}%" >> "$LOG"
fi


#------CHECK_RAM-----
READ=$(free -m | awk 'NR==2 {print $3, $2}')
RAM_USED=$(echo $READ | awk '{print $1}')
RAM_TOTAL=$(echo $READ | awk '{print $2}')
RAM_PCT=$(( $RAM_USED * 100 / $RAM_TOTAL ))
if [ "$RAM_PCT" -gt "$THRESHOLD_RAM" ]; then
        echo "[$TIMESTAMP] [ALERT] Ram: ${RAM_PCT}%" >> "$LOG"
else
        echo "[$TIMESTAMP] [OK] Ram: ${RAM_PCT}%" >> "$LOG"
fi

#-----------CHECK_CPU------
CPU=$(cat /proc/loadavg | awk '{print $1}')
if [ $(echo "$CPU > $THRESHOLD_CPU" | bc) -eq 1 ]; then
        echo "[$TIMESTAMP] [ALERT] CPU load: ${CPU}" >> "$LOG"
else
        echo "[$TIMESTAMP] [OK] CPU load: ${CPU}" >> "$LOG"
fi
```

---

## Giải thích từng block

### CHECK DISK

```bash
DISK=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')
```

- `df -h /` — kiểm tra dung lượng phân vùng root `/`.
- `NR==2 {print $5}` — lấy dòng 2 (bỏ header), cột 5 là `Use%`.
- `tr -d '%'` — xóa ký tự `%` để so sánh số nguyên bằng `-gt`.

---

### CHECK RAM

```bash
READ=$(free -m | awk 'NR==2 {print $3, $2}')
RAM_PCT=$(( $RAM_USED * 100 / $RAM_TOTAL ))
```

- `free -m` — đọc RAM theo MB.
- `NR==2` — lấy dòng `Mem:`, cột 3 (used) và cột 2 (total).
- Tính phần trăm bằng integer arithmetic trong bash — không cần `bc` vì kết quả là số nguyên.

---

### CHECK CPU

```bash
CPU=$(cat /proc/loadavg | awk '{print $1}')
if [ $(echo "$CPU > $THRESHOLD_CPU" | bc) -eq 1 ]
```

- `/proc/loadavg` — đọc load average hệ thống.
- `$1` — giá trị load trung bình **1 phút** gần nhất.
- Dùng `bc` vì load average là **số thực (float)** — toán tử `-gt` trong bash chỉ hỗ trợ số nguyên.

---

## Ví dụ output log

```
[2025-06-09 10:00:01] [OK] Disk: 55%
[2025-06-09 10:00:01] [OK] Ram: 72%
[2025-06-09 10:00:01] [OK] CPU load: 0.85
[2025-06-09 10:05:01] [ALERT] Disk: 83%
[2025-06-09 10:05:01] [ALERT] Ram: 91%
[2025-06-09 10:05:01] [ALERT] CPU load: 3.12
```

---

## Tích hợp với cron

```bash
# Chạy mỗi 5 phút
*/5 * * * * /bin/bash /home/anhtu/health_check.sh
```

---

## Lưu ý

> **Typo trong bản gốc:** `CHECK DISH` và `CHECH_CPU` → đã sửa thành `CHECK DISK` và `CHECK_CPU`.
