# 📄 DOCS — TUẦN 2: Linux Nâng Cao + Git Cơ Bản

> **Deadline:** 05/06/2026 | **Thời gian:** 23h  
> Gồm 2 tài liệu: **Note Git Workflow** + **Cheatsheet Bash Script Template**

---

## 📘 NOTE: Git Workflow Cơ Bản

### Ba vùng của Git

Git quản lý file qua 3 vùng riêng biệt:

| Vùng | Mô tả |
|------|-------|
| **Working Directory** | Nơi bạn chỉnh sửa file thực tế. Git chưa theo dõi thay đổi cho đến khi `git add`. |
| **Staging Area (Index)** | Snapshot chuẩn bị cho commit. `git add` đưa thay đổi vào đây. |
| **Repository (.git)** | Lịch sử commit vĩnh viễn. `git commit` lưu snapshot vào đây. |
| **Remote (GitHub/GitLab)** | Server lưu code online. Dùng `git push` / `git pull` / `git fetch` để sync. |

### Luồng dữ liệu

```
Working Directory  →(git add)→  Staging Area  →(git commit)→  Local Repo  →(git push)→  Remote
Remote  →(git fetch / git pull)→  Local Repo
```

---

### Git Workflow Chuẩn — 6 bước

#### Bước 1 — Clone / Init

```bash
# Clone repo có sẵn về local
git clone https://github.com/user/repo.git

# Hoặc khởi tạo project mới
git init my-project
cd my-project
```

#### Bước 2 — Tạo branch mới

```bash
# Không bao giờ commit thẳng lên main/master
git checkout -b feature/ten-tinh-nang

# Kiểm tra branch hiện tại và tất cả branch
git branch -a
```

> **Quy tắc:** Mỗi tính năng / bug fix = 1 branch riêng.

#### Bước 3 — Chỉnh sửa & Stage

```bash
# Xem trạng thái working tree
git status

# Xem thay đổi chi tiết trước khi stage
git diff

# Stage tất cả thay đổi
git add .

# Stage chỉ 1 file cụ thể
git add src/file.js
```

#### Bước 4 — Commit chuẩn

```bash
# Commit với message theo Conventional Commits
git commit -m "feat: thêm auth module"

# Xem lịch sử commit rút gọn
git log --oneline -5
```

**Conventional Commits — định dạng chuẩn:**

| Prefix | Ý nghĩa |
|--------|---------|
| `feat:` | Thêm tính năng mới |
| `fix:` | Sửa bug |
| `docs:` | Cập nhật tài liệu |
| `chore:` | Task vặt, không ảnh hưởng logic |
| `refactor:` | Cải tiến code, không đổi chức năng |
| `test:` | Thêm / sửa test |

#### Bước 5 — Push & Pull Request

```bash
# Sync với main trước khi push để tránh conflict
git pull origin main --rebase

# Push branch lên remote
git push origin feature/ten-tinh-nang
```

Sau khi push → tạo **Pull Request** trên GitHub để review trước khi merge vào `main`.

#### Bước 6 — Xử lý Conflict

```bash
# Fetch thay đổi mới nhất từ remote
git fetch origin

# Rebase branch hiện tại lên main (giữ lịch sử gọn)
git rebase origin/main

# Sau khi fix conflict trong file, tiếp tục rebase
git add .
git rebase --continue

# Nếu muốn huỷ rebase
git rebase --abort
```

> **Lưu ý:** Dùng `rebase` thay `merge` để giữ lịch sử commit tuyến tính, dễ đọc hơn.

---

### Các lệnh Git thường dùng thêm

```bash
git stash              # Tạm cất thay đổi chưa commit
git stash pop          # Lấy lại thay đổi vừa cất
git reset HEAD~1       # Undo commit cuối (giữ thay đổi trong working dir)
git reset --hard HEAD  # Huỷ toàn bộ thay đổi chưa commit (⚠️ không thể khôi phục)
git tag v1.0.0         # Tạo tag đánh dấu version
```

---

## 📋 CHEATSHEET: Bash Script Template

### Cấu trúc chuẩn của một Bash Script

```bash
#!/bin/bash
# ─────────────────────────────────────────────────────────────
# Script : monitor_disk.sh
# Author : <your-name>
# Date   : $(date +%Y-%m-%d)
# Desc   : Cảnh báo khi disk usage vượt ngưỡng cho phép
# Usage  : ./monitor_disk.sh
# ─────────────────────────────────────────────────────────────

# ── Cấu hình (chỉnh tại đây, không sửa logic bên dưới) ───────
THRESHOLD=80
LOG_FILE="/var/log/disk_monitor.log"
ALERT_EMAIL="admin@example.com"

# ── Helper: ghi log có timestamp ─────────────────────────────
log() {
  echo "[$(date '+%F %T')] $*" >> "$LOG_FILE"
}

# ── Hàm chính: kiểm tra disk ─────────────────────────────────
check_disk() {
  local USAGE
  USAGE=$(df -h / | awk 'NR==2 { gsub(/%/, ""); print $5 }')

  if (( USAGE > THRESHOLD )); then
    log "ALERT: Disk ${USAGE}% (>${THRESHOLD}%) — cần dọn dẹp ngay!"
    # Gửi email cảnh báo (cần mailutils)
    # echo "Disk ${USAGE}%" | mail -s "Disk Alert" "$ALERT_EMAIL"
  else
    log "OK: Disk ${USAGE}% — trong ngưỡng cho phép."
  fi
}

# ── Entry point ───────────────────────────────────────────────
check_disk
```

---

### Syntax Cheatsheet

#### Biến & Input

```bash
NAME="hello"                   # khai báo biến (không có dấu cách quanh =)
echo "$NAME"                   # luôn dùng dấu "" khi tham chiếu biến
readonly PI=3.14               # biến hằng, không thể gán lại

read -p "Nhập tên: " USER_NAME # nhận input từ người dùng

$1 $2 $3                       # tham số truyền vào khi gọi script
$0                             # tên script
$#                             # số lượng tham số
$@                             # tất cả tham số (dạng mảng)
$?                             # exit code của lệnh trước (0 = thành công)
$$                             # PID của script hiện tại
```

#### Điều kiện — `if / elif / else`

```bash
if [ "$A" -gt "$B" ]; then
  echo "A lớn hơn B"
elif [ "$A" -eq "$B" ]; then
  echo "A bằng B"
else
  echo "B lớn hơn A"
fi

# Dạng một dòng
[ -f "$FILE" ] && echo "file tồn tại" || echo "không tồn tại"

# case (thay thế nhiều if-elif)
case "$OPT" in
  start)   echo "Starting..." ;;
  stop)    echo "Stopping..." ;;
  restart) echo "Restarting..." ;;
  *)       echo "Unknown option" ;;
esac
```

**Bảng Test Conditions `[ ]`:**

| Test | Ý nghĩa |
|------|---------|
| `-f file` | File tồn tại (regular file) |
| `-d dir` | Thư mục tồn tại |
| `-e path` | Path tồn tại (file hoặc dir) |
| `-r file` | File có quyền đọc |
| `-z "$V"` | Chuỗi rỗng |
| `-n "$V"` | Chuỗi không rỗng |
| `$A -eq $B` | Bằng nhau (số) |
| `$A -ne $B` | Khác nhau (số) |
| `$A -gt $B` | Lớn hơn (số) |
| `$A -lt $B` | Nhỏ hơn (số) |
| `"$A" = "$B"` | Bằng nhau (chuỗi) |

#### Vòng lặp — `for / while / until`

```bash
# for với range
for i in {1..5}; do
  echo "Lần $i"
done

# for duyệt file
for f in /etc/*.conf; do
  echo "Config: $f"
done

# for kiểu C
for (( i=0; i<5; i++ )); do
  echo "$i"
done

# while
COUNT=0
while [ "$COUNT" -lt 5 ]; do
  echo "$COUNT"
  (( COUNT++ ))
done

# Đọc từng dòng của file
while IFS= read -r line; do
  echo "$line"
done < /etc/hosts
```

#### Functions

```bash
# Khai báo hàm
my_func() {
  local VAR="chỉ tồn tại trong hàm"  # biến local
  echo "Tham số 1: $1"
  return 0                            # 0 = thành công
}

# Gọi hàm
my_func "hello"

# Kiểm tra kết quả
if my_func "test"; then
  echo "Thành công"
fi

# Capture output của hàm
RESULT=$(my_func "arg")
```

---

### Best Practices

```bash
#!/bin/bash
set -e          # thoát ngay khi có lệnh lỗi
set -u          # báo lỗi khi dùng biến chưa khai báo
set -o pipefail # pipe thất bại nếu bất kỳ lệnh nào trong pipe lỗi
# Viết gọn: set -euo pipefail
```

| Quy tắc | Ví dụ đúng | Ví dụ sai |
|---------|-----------|----------|
| Luôn quote biến | `"$VAR"` | `$VAR` |
| Dùng `local` trong hàm | `local X=1` | `X=1` |
| Kiểm tra file tồn tại trước khi dùng | `[ -f "$F" ] && cat "$F"` | `cat $F` |
| Ghi log với timestamp | `echo "[$(date)] msg"` | `echo "msg"` |
| Test dry-run trước khi chạy | `bash -n script.sh` | chạy thẳng |
| Đặt tên biến UPPER_CASE | `THRESHOLD=80` | `threshold=80` |
| Đặt tên hàm lower_case | `check_disk()` | `CheckDisk()` |

---

### Lệnh crontab — kết hợp với script

```bash
# Mở editor chỉnh cron
crontab -e

# Thêm dòng này để chạy script mỗi giờ
0 * * * * /home/user/monitor_disk.sh

# Chạy mỗi ngày lúc 2:00 AM và ghi log
0 2 * * * /home/user/monitor_disk.sh >> /var/log/cron_disk.log 2>&1

# Xem danh sách cron hiện tại
crontab -l
```

---

*Tuần 2 — Linux Nâng Cao + Git Cơ Bản | Tháng 1 — Nền tảng DevOps*
