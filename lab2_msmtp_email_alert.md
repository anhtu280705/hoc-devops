# Lab 2 — Gửi Email Alert với msmtp

> Tích hợp email alert vào monitoring script — khi disk/RAM/CPU vượt ngưỡng, tự động gửi cảnh báo qua Gmail.

---

## 1. Cài đặt msmtp

```bash
sudo apt update && sudo apt install msmtp -y
```

Kiểm tra cài thành công:
```bash
which msmtp
# → /usr/bin/msmtp
```

---

## 2. Tạo App Password cho Gmail

Gmail không cho dùng password thật qua SMTP — phải tạo App Password riêng:

1. Vào [myaccount.google.com](https://myaccount.google.com)
2. Security → **2-Step Verification** → bật lên
3. Security → **App passwords** → tạo mới → đặt tên "msmtp-lab"
4. Google tạo ra 16 ký tự ngẫu nhiên → dùng cái đó làm password

> ⚠️ Không bao giờ dùng password Gmail thật trong config file.

---

## 3. Tạo file config msmtp

```bash
vi ~/.msmtprc
```

Nội dung:

```
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        ~/.msmtp.log

account        gmail
host           smtp.gmail.com
port           587
from           your_email@gmail.com
user           your_email@gmail.com
password       <16_ky_tu_app_password>

account default : gmail
```

Set permission (bắt buộc — msmtp từ chối chạy nếu thiếu):
```bash
chmod 600 ~/.msmtprc
```

---

## 4. Giải thích từng dòng config

| Dòng | Ý nghĩa |
|------|---------|
| `auth on` | Bật xác thực với SMTP server |
| `tls on` | Mã hóa kết nối bằng TLS |
| `tls_trust_file` | File CA để xác minh certificate của Gmail |
| `logfile` | Ghi log mỗi lần gửi mail vào file này |
| `host smtp.gmail.com` | SMTP server của Gmail |
| `port 587` | Port chuẩn cho TLS (STARTTLS) |
| `from` | Địa chỉ hiển thị trong email gửi đi |
| `user` | Tài khoản đăng nhập vào SMTP server |
| `password` | App Password 16 ký tự |
| `account default : gmail` | Dùng account gmail làm mặc định |

> `from` và `user` thường giống nhau với Gmail cá nhân.
> Khác nhau khi dùng mail server công ty (gửi thay mặt địa chỉ khác).

---

## 5. Test gửi mail

```bash
echo "Test alert from maychu" | msmtp your_email@gmail.com
```

Xem log nếu có lỗi:
```bash
cat ~/.msmtp.log
```

---

## 6. Tích hợp vào health_check.sh

Mở script:
```bash
nano ~/opt/monitor/health_check.sh
```

Thêm biến và function sau phần khai báo biến:

```bash
ALERT_EMAIL="your_email@gmail.com"

send_alert() {
    local msg=$1
    echo "$msg" | msmtp "$ALERT_EMAIL"
}
```

Sửa mỗi nhánh ALERT để gọi thêm `send_alert`:

```bash
# --- Check Disk ---
DISK=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')
if [ "$DISK" -gt "$THRESHOLD_DISK" ]; then
    echo "[$TIMESTAMP] [ALERT] Disk: ${DISK}%" >> "$LOG"
    send_alert "[$TIMESTAMP] ALERT: Disk usage ${DISK}% on $(hostname)"
else
    echo "[$TIMESTAMP] [OK]    Disk: ${DISK}%" >> "$LOG"
fi

# --- Check RAM ---
READ=$(free -m | awk 'NR==2 {print $3, $2}')
RAM_USED=$(echo $READ | awk '{print $1}')
RAM_TOTAL=$(echo $READ | awk '{print $2}')
RAM_PCT=$(( RAM_USED * 100 / RAM_TOTAL ))
if [ "$RAM_PCT" -gt "$THRESHOLD_RAM" ]; then
    echo "[$TIMESTAMP] [ALERT] RAM: ${RAM_PCT}%" >> "$LOG"
    send_alert "[$TIMESTAMP] ALERT: RAM usage ${RAM_PCT}% on $(hostname)"
else
    echo "[$TIMESTAMP] [OK]    RAM: ${RAM_PCT}%" >> "$LOG"
fi

# --- Check CPU ---
CPU=$(cat /proc/loadavg | awk '{print $1}')
if [ $(echo "$CPU > $THRESHOLD_CPU" | bc) -eq 1 ]; then
    echo "[$TIMESTAMP] [ALERT] CPU load: ${CPU}" >> "$LOG"
    send_alert "[$TIMESTAMP] ALERT: CPU load ${CPU} on $(hostname)"
else
    echo "[$TIMESTAMP] [OK]    CPU load: ${CPU}" >> "$LOG"
fi
```

---

## 7. Test trigger alert

Tạm hạ threshold để trigger ngay:

```bash
# Sửa trong script:
THRESHOLD_DISK=10    # disk 80% sẽ trigger
THRESHOLD_RAM=10     # RAM 46% sẽ trigger
THRESHOLD_CPU=0.001  # CPU bất kỳ sẽ trigger
```

Chạy thủ công:
```bash
sudo bash ~/opt/monitor/health_check.sh
```

Kiểm tra Gmail nhận được alert, sau đó đổi lại threshold thật:
```bash
THRESHOLD_DISK=80
THRESHOLD_RAM=85
THRESHOLD_CPU=2.0
```

---

## 8. Verify hoàn chỉnh

```bash
# Xem log script
cat ~/opt/monitor/health_check.log

# Xem log msmtp
cat ~/.msmtp.log

# Xem cron đang chạy
sudo crontab -l
grep CRON /var/log/syslog | tail -5
```

---

## Flow tổng thể

```
cron trigger mỗi 5 phút
    → health_check.sh chạy
        → check disk/RAM/CPU
            → nếu vượt ngưỡng: ghi [ALERT] vào log + gửi email qua msmtp
            → nếu bình thường: ghi [OK] vào log
```
