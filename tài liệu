# 📚 Linux Cơ Bản — Tuần 1: Nền Tảng Linux & Network

---

## 1. Hệ Điều Hành Linux — Ubuntu

### 1.1 Linux là gì?

Linux là hệ điều hành mã nguồn mở dựa trên nhân (kernel) Linux, được Linus Torvalds tạo ra năm 1991. **Ubuntu** là bản phân phối (distro) phổ biến nhất, lý tưởng cho người mới bắt đầu và các môi trường server.

### 1.2 Tại sao học Linux?

- Hơn **96% web server** trên thế giới chạy Linux
- Bắt buộc với DevOps, Backend, SysAdmin, Cloud Engineer
- Miễn phí, ổn định, bảo mật cao
- Giao diện dòng lệnh (CLI) mạnh mẽ hơn GUI

### 1.3 Cài Ubuntu trên VMware

**Bước 1:** Tải Ubuntu 22.04 LTS tại [ubuntu.com/download](https://ubuntu.com/download)

**Bước 2:** Tạo VM mới trong VMware Workstation:
- RAM: tối thiểu 2GB (khuyến nghị 4GB)
- Disk: tối thiểu 20GB
- Network: NAT (kết nối internet qua máy host)

**Bước 3:** Boot từ ISO, chọn "Install Ubuntu" → làm theo hướng dẫn

**Bước 4:** Sau cài đặt, cập nhật hệ thống:
```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. Cấu Trúc Thư Mục Linux

Linux tổ chức file theo cây thư mục (Filesystem Hierarchy Standard — FHS), bắt đầu từ `/` (root).

```
/
├── home/       ← Thư mục cá nhân người dùng
│   └── student/
├── etc/        ← File cấu hình hệ thống
├── var/        ← Dữ liệu biến đổi (log, cache)
├── bin/        ← Lệnh cơ bản (ls, cp, mv...)
├── usr/        ← Phần mềm cài thêm
├── tmp/        ← File tạm thời
├── root/       ← Home của user root
├── dev/        ← Thiết bị phần cứng
└── proc/       ← Thông tin tiến trình
```

| Thư mục | Vai trò | Ví dụ |
|---------|---------|-------|
| `/home` | Dữ liệu người dùng | `/home/student/documents` |
| `/etc` | Cấu hình hệ thống | `/etc/nginx/nginx.conf` |
| `/var/log` | File log | `/var/log/syslog` |
| `/bin` | Lệnh hệ thống cơ bản | `/bin/bash`, `/bin/ls` |
| `/usr/bin` | Lệnh phần mềm thêm | `/usr/bin/python3` |
| `/tmp` | File tạm (xóa khi reboot) | Tạo file test tạm thời |

> **Mẹo:** Ký hiệu `~` là viết tắt của `/home/<tên_user>` của bạn.

---

## 3. Lệnh Cơ Bản Linux

### 3.1 Điều hướng thư mục

```bash
# Hiển thị thư mục hiện tại
pwd
# Kết quả: /home/student

# Di chuyển thư mục
cd /etc            # Di chuyển đến /etc
cd ~               # Về thư mục home
cd ..              # Lên thư mục cha
cd -               # Quay lại thư mục trước

# Liệt kê file
ls                 # Liệt kê đơn giản
ls -l              # Liệt kê chi tiết (quyền, kích thước, ngày)
ls -la             # Bao gồm file ẩn (bắt đầu bằng .)
ls -lh             # Kích thước dễ đọc (KB, MB, GB)
```

### 3.2 Quản lý file và thư mục

```bash
# Tạo thư mục
mkdir mydir                   # Tạo 1 thư mục
mkdir -p parent/child/sub     # Tạo nhiều cấp cùng lúc

# Tạo file
touch newfile.txt             # Tạo file rỗng
touch file1.txt file2.txt     # Tạo nhiều file

# Sao chép
cp source.txt backup.txt      # Copy file
cp -r source_dir/ backup_dir/ # Copy thư mục (đệ quy)

# Di chuyển / Đổi tên
mv old.txt new.txt            # Đổi tên file
mv file.txt /tmp/             # Di chuyển file

# Xóa
rm file.txt                   # Xóa file
rm -r mydir/                  # Xóa thư mục đệ quy
rm -rf mydir/                 # Xóa không hỏi ⚠️ Cẩn thận!
```

### 3.3 Xem nội dung file

```bash
cat file.txt           # In toàn bộ nội dung
less file.txt          # Xem từng trang (q để thoát)
head -20 file.txt      # 20 dòng đầu
tail -20 file.txt      # 20 dòng cuối
tail -f /var/log/syslog  # Theo dõi log realtime

# Tìm kiếm trong file
grep "error" file.txt           # Tìm dòng chứa "error"
grep -i "error" file.txt        # Không phân biệt hoa/thường
grep -r "pattern" /var/log/     # Tìm trong toàn bộ thư mục
grep -n "text" file.txt         # Hiển thị số dòng
```

### 3.4 Tìm file

```bash
find / -name "*.conf"           # Tìm file .conf trong toàn hệ thống
find /home -name "*.txt"        # Tìm trong /home
find . -type f -size +1M        # Tìm file lớn hơn 1MB
find /tmp -mtime +7             # Tìm file cũ hơn 7 ngày
```

### 3.5 Thông tin hệ thống

```bash
whoami              # Tên user hiện tại
id                  # UID, GID của user
uname -a            # Thông tin kernel
df -h               # Dung lượng ổ đĩa
free -h             # RAM sử dụng
top                 # Tiến trình đang chạy (q để thoát)
history             # Lịch sử lệnh đã chạy
echo $HOME          # Giá trị biến môi trường HOME
```

---

## 4. Phân Quyền File (Permissions)

### 4.1 Đọc thông tin quyền

```bash
$ ls -l script.sh
-rwxr-xr-x 1 student student 123 Jan 01 10:00 script.sh
```

Cấu trúc phần quyền `-rwxr-xr-x`:

| Vị trí | Ký tự | Ý nghĩa |
|--------|-------|---------|
| 1      | `-`   | Loại: `-` = file, `d` = directory, `l` = symlink |
| 2-4    | `rwx` | **Owner**: đọc, ghi, thực thi |
| 5-7    | `r-x` | **Group**: đọc, thực thi (không ghi) |
| 8-10   | `r-x` | **Others**: đọc, thực thi (không ghi) |

### 4.2 Ký hiệu quyền

| Ký tự | Số | Ý nghĩa |
|-------|----|---------|
| `r`   | 4  | Read — đọc file / liệt kê thư mục |
| `w`   | 2  | Write — ghi, chỉnh sửa |
| `x`   | 1  | Execute — chạy script / vào thư mục |
| `-`   | 0  | Không có quyền |

### 4.3 chmod — Đặt quyền

```bash
# Dùng số (octal)
chmod 755 script.sh    # rwxr-xr-x — script public
chmod 644 config.txt   # rw-r--r-- — file cấu hình
chmod 600 secret.key   # rw------- — file nhạy cảm
chmod 777 file         # rwxrwxrwx — ⚠️ TRÁNH dùng trên server!

# Dùng ký hiệu
chmod +x script.sh     # Thêm quyền execute
chmod -w file.txt      # Bỏ quyền write
chmod u+x,g-w file     # Owner thêm x, group bỏ w

# Áp dụng đệ quy
chmod -R 755 /var/www/
```

### 4.4 chown — Đổi chủ sở hữu

```bash
chown student file.txt            # Đổi owner
chown student:www-data file.txt   # Đổi owner và group
chown -R student /home/student/   # Đệ quy toàn thư mục
```

### 4.5 Bảng quyền phổ biến

| Octal | Symbolic | Dùng cho |
|-------|----------|----------|
| `755` | `rwxr-xr-x` | Script, thư mục public |
| `644` | `rw-r--r--` | File văn bản, HTML, config |
| `700` | `rwx------` | Script cá nhân |
| `600` | `rw-------` | SSH private key, file mật khẩu |
| `664` | `rw-rw-r--` | File chia sẻ trong group |

---

## 5. Soạn Thảo Với Vim

### 5.1 Mở Vim

```bash
vim filename.txt      # Mở file (tạo mới nếu chưa có)
vi filename.txt       # Tương đương trên hầu hết hệ thống
```

### 5.2 Ba chế độ (Mode) của Vi

```
NORMAL ──── i / a ────► INSERT
   ▲                       │
   └────── Esc ────────────┘
   │
   └── : ──► COMMAND (:w, :q, :wq, :s/...)
```

| Chế độ | Kích hoạt | Dùng để |
|--------|-----------|---------|
| **Normal** | `Esc` | Di chuyển, xóa, copy, paste |
| **Insert** | `i`, `a`, `o` | Nhập văn bản |
| **Command** | `:` | Lưu, thoát, tìm kiếm, thay thế |

### 5.3 Lệnh trong Normal Mode

**Di chuyển:**
```
h / l       ← / → (trái / phải)
j / k       ↓ / ↑ (xuống / lên)
0 / $       Đầu / cuối dòng
gg / G      Đầu / cuối file
:50         Nhảy đến dòng 50
w / b       Nhảy theo từ (tiến / lùi)
```

**Chỉnh sửa:**
```
dd          Xóa dòng hiện tại
D           Xóa từ cursor đến cuối dòng
yy          Copy (yank) dòng hiện tại
p           Paste sau cursor
P           Paste trước cursor
u           Undo
x           Xóa 1 ký tự
r<char>     Thay thế 1 ký tự
```

**Insert Mode variants:**
```
i    Chèn trước cursor
a    Chèn sau cursor
o    Mở dòng mới phía dưới
O    Mở dòng mới phía trên
```

### 5.4 Lệnh Command Mode (bắt đầu bằng `:`)

```vi 
:w              Lưu file
:q              Thoát (chỉ khi không có thay đổi)
:q!             Thoát không lưu
:wq             Lưu và thoát
:x              Lưu và thoát (ngắn hơn)
ZZ              Lưu và thoát (không cần :)

/pattern        Tìm "pattern" (n = tiếp, N = lui)
:%s/old/new/g   Thay thế tất cả "old" bằng "new"
:set nu         Bật số dòng
:set nonu       Tắt số dòng
```

### 5.5 Workflow ví dụ

```bash
# Tạo script Hello World bằng vim
vi hello.sh
```

Trong vi:
1. Nhấn `i` → chuyển sang INSERT mode
2. Gõ nội dung:
   ```bash
   #!/bin/bash
   echo "Hello, Linux!"
   ```
3. Nhấn `Esc` → về NORMAL mode
4. Gõ `:wq` → lưu và thoát

```bash
# Cấp quyền thực thi và chạy
chmod 755 hello.sh
./hello.sh
# Kết quả: Hello, Linux!
```

---

## 6. Thực Hành / Project Tuần 1

### Task 1 — Cài Ubuntu trên VMware

- [ ] Tải Ubuntu 22.04 LTS Desktop/Server
- [ ] Tạo VM trong VMware (RAM 2GB+, Disk 20GB+)
- [ ] Hoàn tất cài đặt và đăng nhập
- [ ] Chạy `sudo apt update && sudo apt upgrade -y`
- [ ] Xác nhận: `uname -a` và `lsb_release -a`

### Task 2 — Thực hành 15 lệnh Linux

```bash
# Tạo cấu trúc thư mục
mkdir -p ~/project/{docs,scripts,logs}

# Tạo và thao tác file
touch ~/project/docs/{readme.txt,notes.txt,config.txt}
echo "Hello World" > ~/project/docs/readme.txt
cat ~/project/docs/readme.txt

# Copy và di chuyển
cp ~/project/docs/readme.txt ~/project/docs/readme_backup.txt
mv ~/project/docs/notes.txt ~/project/logs/

# Tìm kiếm
grep "Hello" ~/project/docs/readme.txt
find ~/project -name "*.txt"

# Xem thông tin
ls -la ~/project/
df -h
free -h
```

### Task 3 — File, Phân quyền, Vi

```bash
# Tạo script bằng vi
vi ~/project/scripts/greet.sh
# Nội dung:
# #!/bin/bash
# echo "Chào mừng, $(whoami)!"
# date

# Phân quyền và chạy
chmod 755 ~/project/scripts/greet.sh
./~/project/scripts/greet.sh

# Tạo file config với quyền giới hạn
vim ~/project/docs/secret.conf
# Nội dung: db_password=mypassword123
chmod 600 ~/project/docs/secret.conf

# Kiểm tra quyền
ls -la ~/project/scripts/
ls -la ~/project/docs/
```

## 7. Tài Liệu Tham Khảo

| Nguồn | Mô tả | Link |
|-------|-------|------|
| LinuxCommand.org | Hướng dẫn chi tiết cho người mới | [linuxcommand.org](http://linuxcommand.org) |
| man pages | Tài liệu chính thức ngay trong terminal: `man ls` | Built-in |
| tldr.sh | Tóm tắt lệnh ngắn gọn, có ví dụ | [tldr.sh](https://tldr.sh) |
| Ubuntu Docs | Tài liệu chính thức Ubuntu | [ubuntu.com/server/docs](https://ubuntu.com/server/docs) |

---

