# Bash Script — Tài liệu cú pháp Phase 1

> Tham chiếu đầy đủ: Biến, If/Else, Vòng lặp, Hàm, Bảng tra nhanh.

---

## 1. Biến

### Khai báo biến

Bash không có kiểu dữ liệu — mọi biến đều là chuỗi. Gán biến bằng `=`, đọc biến bằng `$`.

```bash
# ✅ Đúng
NAME="nginx"
PORT=8080
MSG="hello world"
```

```bash
# ❌ Sai — có dấu cách quanh dấu =
NAME = "nginx"    # LỖI
PORT =8080        # LỖI
MSG= "hello"      # LỖI
```

> ❌ **Quy tắc số 1:** `KHÔNG bao giờ có dấu cách quanh dấu =` khi gán biến.  
> Bash sẽ hiểu `NAME` là lệnh, `=` và `"nginx"` là tham số.

---

### Đọc và dùng biến

```bash
NAME="nginx"

echo "$NAME"           # cách thường
echo "${NAME}"         # cách rõ ràng — khuyên dùng
echo "${NAME}-proxy"   # PHẢI dùng {} khi ghép chuỗi
echo "$NAME-proxy"     # vẫn đúng nhưng dễ nhầm
echo "$NAMEproxy"      # SAI — bash tìm biến $NAMEproxy
```

> 💡 Thói quen tốt: luôn dùng `${VAR}` thay vì `$VAR` — tránh bug khó tìm khi ghép chuỗi.

---

### Các loại biến đặc biệt

| Biến | Ý nghĩa | Ví dụ |
|------|---------|-------|
| `$0` | Tên script đang chạy | `./deploy.sh` → `$0 = "./deploy.sh"` |
| `$1` `$2` ... | Tham số truyền vào (argument) | `./deploy.sh web 3000` → `$1="web"`, `$2="3000"` |
| `$@` | Tất cả tham số (giữ nguyên từng cái) | `for x in "$@"` để duyệt an toàn |
| `$#` | Số lượng tham số | `$# -eq 0` → không có tham số nào |
| `$?` | Exit code của lệnh vừa chạy | `0` = thành công, khác `0` = lỗi |
| `$$` | PID của script hiện tại | Tạo file tạm: `/tmp/lock.$$` |
| `$USER` | Tên user đang chạy | `root`, `ubuntu`, `deploy`... |
| `$HOME` | Thư mục home | `/root`, `/home/ubuntu` |
| `$PWD` | Thư mục hiện tại | `/var/www/app` |

---

### Giá trị mặc định — cú pháp `${VAR:-default}`

```bash
# ${VAR:-default} = dùng VAR nếu có, không thì dùng default
ENV="${1:-production}"
PORT="${APP_PORT:-8080}"
HOST="${SERVER_HOST:-localhost}"

# ${VAR:?msg} = dừng script và báo lỗi nếu VAR rỗng
API_KEY="${API_KEY:?Lỗi: cần set biến API_KEY}"
```

> ✅ Trong DevOps, `${1:-localhost}` rất hay dùng để script vừa nhận tham số vừa có giá trị mặc định an toàn — tránh lỗi khi quên truyền argument.

---

### Command substitution — nhúng kết quả lệnh vào biến

```bash
TODAY=$(date +%Y-%m-%d)
FREE_MEM=$(free -m | awk 'NR==2{print $4}')
MY_IP=$(hostname -I | awk '{print $1}')

echo "Ngày: $TODAY | IP: $MY_IP | RAM trống: ${FREE_MEM}MB"
```

> `$(lệnh)` chạy lệnh trong ngoặc và thay thế bằng output. Đây là cú pháp hiện đại — tránh dùng backtick `` `lệnh` `` (cũ, khó đọc, không lồng được).

---

## 2. If / Else

### Cấu trúc if — bộ khung cơ bản

```bash
if [[ ĐIỀU_KIỆN ]]; then
    # chạy khi điều kiện đúng
elif [[ ĐIỀU_KIỆN_2 ]]; then
    # chạy khi điều kiện 2 đúng
else
    # chạy khi tất cả đều sai
fi   # bắt buộc — đóng khối if
```

> ❌ Luôn có dấu cách bên trong `[[ ]]`. Viết `[[$X -gt 0]]` sẽ báo lỗi — phải là `[[ $X -gt 0 ]]`.

---

### `[ ]` vs `[[ ]]` — dùng cái nào?

```bash
# ❌ [ ] — cũ, dễ lỗi khi biến có dấu cách
F="my file"
[ -f $F ]        # LỖI! (bị split thành 2 argument)
[ -f "$F" ]      # phải quote thủ công

# Không dùng được && || bên trong
[ $A -gt 1 -a $B -lt 5 ]   # phải dùng -a thay cho &&
```

```bash
# ✅ [[ ]] — hiện đại, an toàn hơn
F="my file"
[[ -f $F ]]       # OK — tự xử lý dấu cách

# Dùng được && || bên trong
[[ $A -gt 1 && $B -lt 5 ]]

# Hỗ trợ regex
[[ $IP =~ ^[0-9] ]]
```

> ✅ Quy tắc đơn giản: **luôn dùng `[[ ]]`** trong bash script. Chỉ dùng `[ ]` khi cần tương thích với `sh` (không phải bash).

---

### Toán tử so sánh — số nguyên

| Toán tử | Nghĩa | Ví dụ |
|---------|-------|-------|
| `-eq` | bằng (equal) | `[[ $A -eq $B ]]` |
| `-ne` | khác (not equal) | `[[ $A -ne 0 ]]` |
| `-gt` | lớn hơn (greater than) | `[[ $CPU -gt 80 ]]` |
| `-lt` | nhỏ hơn (less than) | `[[ $PORT -lt 1024 ]]` |
| `-ge` | lớn hơn hoặc bằng | `[[ $RETRY -ge 3 ]]` |
| `-le` | nhỏ hơn hoặc bằng | `[[ $i -le 10 ]]` |

> ⚠️ Không dùng `>` `<` để so sánh số — bash sẽ hiểu đó là redirect file. Chỉ dùng `-gt` `-lt` cho số nguyên.

---

### Toán tử kiểm tra chuỗi & file

**Chuỗi:**

| Toán tử | Ý nghĩa |
|---------|---------|
| `-z "$s"` | chuỗi rỗng |
| `-n "$s"` | chuỗi không rỗng |
| `==` hoặc `=` | chuỗi bằng nhau |
| `!=` | chuỗi khác nhau |
| `=~` | khớp regex (chỉ trong `[[ ]]`) |

**File / thư mục:**

| Toán tử | Ý nghĩa |
|---------|---------|
| `-f file` | file tồn tại (là file thường) |
| `-d dir` | thư mục tồn tại |
| `-e path` | tồn tại (bất kỳ loại) |
| `-r file` | có quyền đọc |
| `-x file` | có quyền thực thi |

---

### Viết tắt — `&&` và `||`

```bash
# Thay vì if dài dòng, dùng && || để viết ngắn gọn

# Tạo thư mục nếu chưa có
[[ -d "/tmp/deploy" ]] || mkdir -p /tmp/deploy

# Chạy lệnh, nếu lỗi thì thoát
git pull || { echo "git pull thất bại"; exit 1; }

# Chỉ chạy lệnh 2 khi lệnh 1 thành công
nginx -t && systemctl reload nginx

# Logic: A && B || C  =  nếu A thì B, không thì C
[[ $DISK -lt 80 ]] && echo "OK" || echo "Cảnh báo"
```

---

## 3. Vòng lặp

### For — 3 kiểu cú pháp

```bash
# Kiểu 1: lặp qua danh sách
for VAR in item1 item2 item3; do
    echo "$VAR"
done

# Kiểu 2: C-style (giống C/Java)
for (( i=1; i<=10; i++ )); do
    echo "Lần $i"
done

# Kiểu 3a: lặp qua kết quả lệnh — TRÁNH DÙNG (dễ lỗi khi tên file có dấu cách)
for FILE in $(ls /var/log/*.log); do
    echo "Xử lý: $FILE"
done

# Kiểu 3b: cách chuẩn — dùng glob trực tiếp
for FILE in /var/log/*.log; do
    echo "Xử lý: $FILE"
done
```

> ✅ Ưu tiên glob (`/var/log/*.log`) thay vì `$(ls ...)` — an toàn với tên file có dấu cách, hiệu quả hơn.

---

### While — 3 pattern hay dùng nhất

```bash
# Pattern 1: vòng lặp vô hạn (monitor / daemon)
while true; do
    echo "$(date): đang chạy..."
    sleep 5
done

# Pattern 2: đọc từng dòng file
while read -r LINE; do
    echo "Xử lý: $LINE"
done < /etc/hosts

# Pattern 3: retry với đếm
COUNT=0
while ! curl -sf http://localhost/health; do
    (( COUNT++ ))
    [[ $COUNT -ge 10 ]] && { echo "Timeout"; exit 1; }
    sleep 3
done
```

> ✅ `while read -r LINE; do ... done < file` là cách chuẩn đọc file theo dòng.  
> Cờ `-r` ngăn bash xử lý ký tự `\` trong dữ liệu (raw read).

---

### `break` và `continue`

```bash
# break — thoát vòng lặp ngay
for SERVER in web01 web02 web03; do
    [[ $SERVER == "web02" ]] && break
    echo "$SERVER"   # chỉ in web01
done

# continue — bỏ qua iteration hiện tại
for PORT in 80 443 8080 8443; do
    [[ $PORT -lt 1024 ]] && continue   # bỏ qua 80, 443
    echo "Port cao: $PORT"             # chỉ in 8080, 8443
done
```

---

### Tính toán số học

```bash
# Dùng (( )) cho phép tính — KHÔNG dùng expr (cũ, chậm)
A=10
B=3

SUM=$(( A + B ))      # 13
DIV=$(( A / B ))      # 3 (chia nguyên — bash không tính số thực)
MOD=$(( A % B ))      # 1 (chia lấy dư)
(( COUNT++ ))         # tăng lên 1
(( COUNT += 5 ))      # cộng thêm 5

# Bash không tính số thực — dùng bc hoặc awk
RESULT=$(echo "scale=2; 10/3" | bc)   # 3.33
```

> ⚠️ Bash chỉ tính số nguyên. Khi cần số thực (ví dụ tính % RAM) dùng `bc` hoặc `awk`.

---

## 4. Hàm

### Khai báo hàm — 2 cách viết

```bash
# Cách 1 — có từ khóa function
function ten_ham() {
    # thân hàm
}

# Cách 2 — không có function (phổ biến hơn, khuyên dùng)
ten_ham() {
    # thân hàm
}
```

> Cả hai đều đúng. Cách 2 được dùng nhiều hơn trong thực tế vì ngắn gọn hơn.

> ❌ Hàm phải được **khai báo trước khi gọi**. Bash đọc từ trên xuống — gọi hàm trước khi định nghĩa sẽ báo lỗi `command not found`.

---

### Tham số trong hàm

```bash
deploy() {
    local SERVER="$1"                    # tham số 1
    local PORT="$2"                      # tham số 2
    local ENV="${3:-production}"         # tham số 3, mặc định "production"

    echo "Deploy lên $SERVER:$PORT môi trường $ENV"
}

# Gọi hàm — truyền tham số cách nhau bằng dấu cách
deploy "web01" 3000
deploy "web02" 3000 "staging"
```

> Bên trong hàm, `$1 $2...` là tham số của **hàm**, không phải tham số của script. `$0` vẫn là tên script.

---

### `local` — biến cục bộ

```bash
# ❌ Không dùng local — biến global bị ghi đè
NAME="global"
foo() {
    NAME="bên trong hàm"
    echo "$NAME"
}
foo
echo "$NAME"   # In ra: "bên trong hàm" — BUG!
```

```bash
# ✅ Dùng local — an toàn
NAME="global"
foo() {
    local NAME="bên trong hàm"
    echo "$NAME"
}
foo
echo "$NAME"   # In ra: "global" — đúng
```

> ✅ Quy tắc: **mọi biến bên trong hàm đều phải có `local`**. Không có ngoại lệ. Đây là thói quen phân biệt người mới với người có kinh nghiệm.

---

### Return value — trả về giá trị

```bash
# Cách 1: hàm trả về exit code (0–255) để dùng trong if
is_running() {
    systemctl is-active --quiet "$1"
    # Không cần "return $?" — bash tự dùng exit code của lệnh cuối
}

if is_running nginx; then
    echo "nginx OK"
fi

# Cách 2: echo để trả chuỗi, nhận bằng $()
get_ip() {
    hostname -I | awk '{print $1}'
}

MY_IP=$(get_ip)
echo "IP: $MY_IP"
```

> ⚠️ `return` chỉ trả được số `0–255` (exit code). Để trả chuỗi, dùng `echo` bên trong hàm và nhận bằng `$(ten_ham)`.  
> Không cần `return $?` sau lệnh cuối — bash tự dùng exit code của lệnh cuối làm return value.

---

### Pattern hàm chuẩn DevOps

```bash
#!/bin/bash
set -euo pipefail

# Hàm utility — khai báo đầu tiên
log()   { echo "[$(date '+%H:%M:%S')] [INFO]  $*"; }
warn()  { echo "[$(date '+%H:%M:%S')] [WARN]  $*" >&2; }
error() { echo "[$(date '+%H:%M:%S')] [ERROR] $*" >&2; exit 1; }

# Hàm business logic
check_root() {
    [[ $EUID -ne 0 ]] && error "Script cần chạy với quyền root"
    log "Đang chạy với quyền root"
}

# Hàm main — gọi cuối cùng
main() {
    check_root
    log "Bắt đầu..."
}

main "$@"   # gọi main, truyền toàn bộ tham số vào
```

> `set -euo pipefail` là dòng thứ 2 bắt buộc của mọi production script:  
> `-e` = thoát khi có lỗi, `-u` = lỗi khi dùng biến chưa khai báo, `-o pipefail` = pipeline lỗi nếu bất kỳ lệnh nào thất bại.

---

## 5. Bảng tra nhanh

| Tình huống | Cú pháp |
|------------|---------|
| Gán biến | `NAME="value"` — không có dấu cách |
| Đọc biến | `${NAME}` — dùng `{}` khi ghép chuỗi |
| Giá trị mặc định | `${VAR:-default}` |
| Bắt buộc có giá trị | `${VAR:?thông báo lỗi}` |
| Nhúng kết quả lệnh | `VAR=$(lệnh)` |
| Tính toán số | `$(( A + B ))` hoặc `(( COUNT++ ))` |
| If đơn giản | `if [[ ĐIỀU_KIỆN ]]; then ... fi` |
| Kiểm tra số | `-eq` `-ne` `-gt` `-lt` `-ge` `-le` |
| Kiểm tra chuỗi | `-z` (rỗng) `-n` (không rỗng) `==` `!=` |
| Kiểm tra file | `-f` (file) `-d` (dir) `-e` (tồn tại) |
| Viết tắt nếu-thì | `command && echo OK \|\| echo FAIL` |
| For danh sách | `for x in a b c; do ... done` |
| For số | `for (( i=0; i<10; i++ )); do ... done` |
| While đọc file | `while read -r line; do ... done < file` |
| Khai báo hàm | `ten_ham() { ... }` |
| Biến cục bộ | `local VAR="value"` — bên trong hàm |
| Gọi hàm | `ten_ham arg1 arg2` |
| Trả exit code | `return 0` hoặc `return 1` |
| Trả chuỗi từ hàm | `echo "kết quả"` → nhận bằng `$(ten_ham)` |
| Dừng khi có lỗi | `set -euo pipefail` — dòng 2 của mọi script |
| Redirect stdout | `lệnh > file` (ghi đè) / `lệnh >> file` (nối) |
| Redirect stderr | `lệnh 2> file` |
| Gộp stdout + stderr | `lệnh >> file 2>&1` |
| Nuốt toàn bộ output | `lệnh > /dev/null 2>&1` |

---

## Phụ lục — Các lỗi đã sửa so với bản gốc

| # | Vị trí | Vấn đề gốc | Đã sửa thành |
|---|--------|-----------|--------------|
| 1 | Hàm `is_running` | `return $?` thừa sau lệnh cuối | Bỏ `return $?` — bash tự dùng exit code của lệnh cuối |
| 2 | Hàm `get_ip` | `$( get_ip )` có dấu cách không chuẩn | `$(get_ip)` — style chuẩn |
| 3 | For loop | `$(ls /var/log/*.log)` thiếu cảnh báo rõ | Đánh dấu rõ là anti-pattern, glob là chuẩn |
| 4 | Ghi chú `set -euo pipefail` | Không giải thích từng flag | Thêm giải thích `-e`, `-u`, `-o pipefail` |
