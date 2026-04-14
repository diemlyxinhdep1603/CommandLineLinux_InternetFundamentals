# Báo cáo Linux Command Line 


# A. PING & HPING3
## I. Lệnh Ping (Packet Internet Groper)

### 1. Khái niệm cơ bản
- **Ping** là một công cụ quản trị mạng cơ bản trên hệ điều hành Linux, được sử dụng phổ biến để kiểm tra kết nối (tính khả dụng) giữa hai thiết bị trên mạng.
- Lệnh Ping hoạt động dựa trên giao thức **ICMP (Internet Control Message Protocol)** bằng cách gửi các gói tin *Echo Request* đến máy chủ đích và chờ đợi máy chủ đó gửi lại gói tin *Echo Reply*.

### 2. Cú pháp và Các tham số phổ biến
**Cú pháp cơ bản:**
```bash
ping [tham_số] [tên_miền_hoặc_IP]
```

**Các tham số thường dùng:**
- `-c [số_lượng]`: Chỉ định số lượng gói tin ICMP sẽ gửi đi thay vì gửi liên tục (Ví dụ: `ping -c 4`).
- `-i [giây]`: Thiết lập thời gian chờ (interval) giữa các lần gửi gói tin (Mặc định là gửi 1 gói mỗi giây).
- `-s [kích_thước]`: Thay đổi kích thước (bytes) của gói tin gửi đi.
- `-W [giây]`: Thời gian chờ tối đa cho một phản hồi từ máy chủ (Timeout).

### 3. Thực hành và Phân tích kết quả
**Thực thi lệnh kiểm tra tới máy chủ Vietnix:**

```bash
ping -c 4 vietnix.vn
```

**Kết quả hiển thị (Output mẫu):**
```bash
lyhuynh@Huynhs-Mac-mini ~ % ping -c 4 vietnix.vn
PING vietnix.vn (14.225.253.240): 56 data bytes
64 bytes from 14.225.253.240: icmp_seq=0 ttl=53 time=5.298 ms
64 bytes from 14.225.253.240: icmp_seq=1 ttl=53 time=11.518 ms
64 bytes from 14.225.253.240: icmp_seq=2 ttl=53 time=5.153 ms
64 bytes from 14.225.253.240: icmp_seq=3 ttl=53 time=11.665 ms

--- vietnix.vn ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 5.153/8.409/11.665/3.184 ms
```

**Giải thích các thông số:**
- **ttl (Time To Live):** Thời gian sống của gói tin. Giá trị này tự động giảm đi 1 sau mỗi lần đi qua một thiết bị định tuyến (Router). Khi TTL = 0, gói tin sẽ bị hủy để tránh tình trạng tắc nghẽn (loop) vô hạn trên mạng.
- **time:** Thời gian trễ (Latency/Round Trip Time) tính bằng mili-giây (ms). Là tổng thời gian từ lúc gửi gói tin đến khi nhận được phản hồi.

---

## II. Lệnh Hping3

### 1. Khái niệm cơ bản
- **Hping3** là một công cụ dòng lệnh nâng cao được dùng để tạo, chỉnh sửa và phân tích các gói tin TCP/IP. 
- Điểm vượt trội của `hping3` so với `ping` truyền thống là nó hỗ trợ linh hoạt nhiều giao thức như **TCP, UDP, ICMP và RAW-IP**.

### 2. Sự khác biệt ứng dụng so với Ping
- **Khả năng xuyên tường lửa (Firewall):** Hiện nay, nhiều hệ thống cấu hình Firewall chặn hoàn toàn gói tin ICMP để bảo mật (khiến lệnh `ping` thất bại dù server vẫn sống). `hping3` giải quyết vấn đề này bằng cách gửi gói tin TCP (vào Port 80 hoặc 443) để "đánh lừa" Firewall, giúp kỹ thuật viên xác định chính xác tình trạng của máy chủ.

### 3. Cú pháp và Các tham số phổ biến
**Cú pháp cơ bản:**
```bash
hping3 [tham_số] [tên_miền_hoặc_IP]
```

**Các tham số thường dùng:**
- `-S`: Gửi gói tin TCP mang cờ SYN (Yêu cầu kết nối).
- `-p [port]`: Chỉ định cổng đích muốn kiểm tra (Ví dụ: `-p 80` cho HTTP).
- `-c [số_lượng]`: Số lượng gói tin sẽ gửi đi.
- `--flood`: Gửi gói tin với tốc độ nhanh nhất có thể. (Khuyến cáo: Chỉ dùng khi test hiệu năng hệ thống của chính mình, không dùng lên domain công cộng vì đây là hình thức tấn công DoS).

### 4. Ví dụ ứng dụng thực tế
**Mô phỏng kiểm tra cổng 80 (HTTP) của Server xem có đang mở hay không bằng TCP SYN:**
```bash
hping3 -S -p 80 -c 4 vietnix.vn
```

**Kết quả hiển thị:**
```bash
HPING vietnix.vn (eth0 103.200.23.123): S set, 40 headers + 0 data bytes
len=46 ip=103.200.23.123 ttl=54 id=42911 sport=80 flags=SA seq=0 win=29200 rtt=14.5 ms
len=46 ip=103.200.23.123 ttl=54 id=42912 sport=80 flags=SA seq=1 win=29200 rtt=15.1 ms
```
*Giải thích:* Thông số quan trọng nhất ở đây là `flags=SA` (SYN-ACK). Khi nhận được cờ SA, điều này chứng minh cổng 80 trên Server Vietnix đang mở và hoạt động bình thường, sẵn sàng nhận kết nối từ người dùng.
# B. LỆNH QUẢN TRỊ TỪ XA (SSH COMMAND)

## I. SSH (Secure Shell) là gì?
- **SSH** là một giao thức mạng mã hóa, được sử dụng để kết nối và quản trị các máy chủ (Server/VPS) từ xa một cách an toàn. Khác với các giao thức cũ (như Telnet), mọi dữ liệu truyền qua SSH đều được mã hóa mạnh mẽ để chống lại việc nghe lén trên mạng.


## II. Các phương thức kết nối SSH

### 1. Kết nối bằng Password 
Đây là phương pháp cơ bản và phổ biến nhất khi nhà cung cấp vừa khởi tạo xong VPS/Server cho khách hàng.
- **Cú pháp:**
```bash
ssh [username]@[IP_Server]
```
- **Ví dụ:** `ssh root@103.200.23.123`
- **Lưu ý:** Khi hệ thống yêu cầu nhập mật khẩu (`password:`), màn hình Terminal sẽ **không hiển thị bất kỳ ký tự hay dấu sao (*)** nào để bảo mật. Chỉ cần gõ đúng mật khẩu và nhấn Enter.

### 2. Kết nối bằng SSH Key (Khóa bảo mật)
Đây là phương thức xác thực an toàn và chuyên nghiệp nhất, thay thế hoàn toàn việc gõ mật khẩu. Nó sử dụng một cặp khóa: *Public Key* (đặt trên Server) và *Private Key*.

- **Cú pháp:** Sử dụng tham số `-i` (Identity file) để trỏ tới đường dẫn chứa file Private Key.
```bash
ssh -i [đường_dẫn_đến_file_private_key] [username]@[IP_Server]
```
- **Ví dụ:**
```bash
ssh -i ~/.ssh/vietnix_key.pem root@103.200.23.123
```
- **Lưu ý:** Nếu file key có quyền cho phép người khác đọc (quá lỏng lẻo), SSH sẽ từ chối kết nối. Phải set quyền read-only cho file này trước khi dùng:
```bash
chmod 400 ~/.ssh/vietnix_key.pem
```

### 3. Kết nối bằng Port Custom (Cổng tùy chỉnh)
Mặc định, dịch vụ SSH hoạt động trên cổng (Port) **22**. Tuy nhiên, để phòng tránh các tool quét mật khẩu tự động (Brute-force attack), các hệ thống thường đổi sang một Port khác (ví dụ: 2222, 2026).
- **Cú pháp:** Sử dụng tham số `-p` để chỉ định Port.
```bash
ssh -p [số_Port] [username]@[IP_Server]
```
- **Ví dụ:**
```bash
ssh -p 2222 root@103.200.23.123
```

### 4. Kết hợp Key và Custom Port (Cấu hình phổ biến thực tế)
Trong môi trường thực tế, hệ thống thường được bảo mật lớp kép: Vừa đổi Port, vừa yêu cầu đăng nhập bằng Key.Có thể kết hợp các tham số lại với nhau:
```bash
ssh -i ~/.ssh/vietnix_key.pem -p 2222 root@103.200.23.123
```
```


# C. LỆNH SAO CHÉP DỮ LIỆU BẢO MẬT (SCP & RSYNC)

## I. Lệnh SCP (Secure Copy Protocol)

### 1. Khái niệm cơ bản
- **SCP** là lệnh dùng để truyền tải file và thư mục một cách bảo mật giữa máy tính cá nhân (Local) và máy chủ (Remote), hoặc giữa hai máy chủ với nhau.
- Điểm mạnh của SCP là nó hoạt động hoàn toàn dựa trên giao thức SSH. Điều này có nghĩa là toàn bộ dữ liệu truyền tải đều được mã hóa, đảm bảo an toàn tuyệt đối khỏi việc bị đánh cắp hay đọc trộm giữa chừng.

### 2. Copy 1 File (Tệp tin)
Cú pháp của SCP luôn tuân theo quy tắc logic: `scp [Nguồn] [Đích]`

**Kịch bản 1: Đẩy file từ máy cá nhân LÊN Server (Upload):**
```bash
scp [đường_dẫn_local]/file.txt [username]@[IP_Server]:[đường_dẫn_đích]
```
*Ví dụ:* Đẩy file có tên Topic.md từ Documents lên thư mục `/root/` của Server:

```bash
scp ~/Documents/Topic.md root@103.200.23.123:/root/
```

**Kịch bản 2: Kéo file từ Server VỀ máy cá nhân (Download):**
```bash
scp [username]@[IP_Server]:[đường_dẫn_file_trên_server] [đường_dẫn_local_lưu_file]
```
*Ví dụ:* Kéo file log lỗi từ Server về máy local để phân tích:
```bash
scp root@103.200.23.123:/var/log/nginx/error.log ~/Desktop/
```

### 3. Copy 1 Folder (Thư mục)
Để copy một thư mục (bao gồm tất cả file và thư mục con bên trong),  **bắt buộc** phải sử dụng thêm tham số `-r` (recursive - đệ quy).

**Cú pháp đẩy thư mục lên Server:**
```bash
scp -r [đường_dẫn_thư_mục_local] [username]@[IP_Server]:[đường_dẫn_đích_trên_server]

- Ví dụ
```bash
scp -r ~/Desktop/Source_Code/ root@103.200.23.123:/var/www/html/
```

**Cú pháp kéo thư mục về máy:**
```bash
scp -r [username]@[IP_Server]:[đường_dẫn_thư_mục_trên_server] [đường_dẫn_local_lưu_thư_mục]
```
- Ví dụ:
```bash
scp -r root@103.200.23.123:/var/www/html/backup_web/ ~/Desktop/
```

### 4. Lưu ý 
- **Custom Port: ** Nếu Server sử dụng Port SSH tùy chỉnh (ví dụ 2222), phải dùng tham số `-P` (**Chữ P viết hoa**). Lệnh `ssh` dùng `-p` viết thường, nhưng `scp` bắt buộc dùng `-P` viết hoa.
  *Ví dụ:* `scp -P 2222 file.txt root@103.200.23.123:/root/`
- **Xác thực bằng Key:** Vì SCP chạy qua SSH, nếu Server cấu hình đăng nhập bằng Key, phải chỉ định đường dẫn file Key bằng tham số `-i` giống hệt lệnh SSH:
  *Ví dụ:* `scp -i ~/.ssh/key.pem file.txt root@103.200.23.123:/root/`

 khi cần chuyển dữ liệu website cho khách hàng, vì nó có khả năng kết nối lại khi rớt mạng và chỉ copy những gì bị thay đổi.

## II. LỆNH ĐỒNG BỘ DỮ LIỆU NÂNG CAO (RSYNC)

### 1. Tổng quan về Rsync
- **Rsync (Remote Sync)** là công cụ dùng để sao chép và đồng bộ hóa tệp tin/thư mục cục bộ (local) hoặc qua mạng (remote). 
- Khác với `scp` (copy toàn bộ mọi thứ từ đầu đến cuối), `rsync` thông minh hơn rất nhiều nhờ thuật toán **Delta-transfer**: Nó sẽ so sánh Nguồn và Đích, sau đó chỉ gửi đi những phần dữ liệu có sự thay đổi.
- Ví dụ: Trong quá trình chuyển dữ liệu website cho khác hàng, nếu trong quá trình chuyển bị rớt mạng thì `rcp` có khả năng kết nối lại và chỉ copy những gì bị thay đổi.
- **Các tham số luôn đi kèm nhau: `-avz`**
  - `-a` (archive): Giữ nguyên mọi thuộc tính của file (quyền, chủ sở hữu, thời gian tạo).
  - `-v` (verbose): Hiển thị chi tiết quá trình copy ra màn hình.
  - `-z` (compress): Nén dữ liệu trong quá trình truyền để tăng tốc độ.

### 2. Cách sử dụng Rsync

#### a. Copy 1 File (Tệp tin)
- **Cú pháp tổng quát:**
```bash
rsync -avz [đường_dẫn_file_nguồn] [username]@[IP_Server]:[đường_dẫn_đích]
```
- **Ví dụ minh họa:** Kéo file database từ Server về máy cá nhân:
```bash
rsync -avz root@103.200.23.123:/root/database.sql ~/Desktop/
```

#### b. Copy 1 Folder (Thư mục)
**Lưu ý về dấu gạch chéo (`/`) ở thư mục nguồn:**
- Nếu nguồn là `thu_muc/` (có dấu gạch chéo): Chỉ copy **nội dung bên trong** thư mục đó.
- Nếu nguồn là `thu_muc` (không có dấu gạch): Copy **cả cái vỏ thư mục** và nội dung bên trong.

- **Cú pháp tổng quát:**
```bash
rsync -avz [đường_dẫn_thư_mục_nguồn] [username]@[IP_Server]:[đường_dẫn_đích]
```
- **Ví dụ minh họa:** Copy toàn bộ source code web (cả vỏ folder `public_html`) lên Server:

```bash
rsync -avz ~/Desktop/public_html root@103.200.23.123:/var/www/
```

### 3. Rsync Incremental (Đồng bộ tăng dần)
- **Bản chất:** Chức năng *Incremental* (tăng dần) là tính năng mặc định làm nên tên tuổi của `rsync`. Nếu quá trình copy bị đứt mạng giữa chừng, hoặc chạy lại lệnh `rsync` một lần nữa, nó sẽ không copy lại từ đầu. Nó chỉ quét xem file nào mới được thêm vào hoặc file nào bị sửa đổi thì mới tiến hành chuyển đi.
- **Kịch bản thực tế (Mirroring):** Khi đồng bộ tăng dần, đôi khi chúng ta xóa một file ở Nguồn, và chúng ta cũng muốn file đó biến mất ở Đích. Lúc này, cần thêm tham số `--delete`.

- **Cú pháp đồng bộ tăng dần chính xác tuyệt đối (Mirror):**
```bash
rsync -avz --delete [Nguồn] [Đích]
```
- **Ví dụ minh họa:** Đồng bộ thư mục backup từ Server về máy cá nhân (nếu trên Server xóa file backup cũ, máy cá nhân cũng tự động xóa theo để giải phóng ổ cứng):
```bash
rsync -avz --delete root@103.200.23.123:/backup/ ~/Desktop/local_backup/
```


# D. NHÓM 1: XEM NỘI DUNG TỆP TIN (VIEW)

## 1. Cat Command
- **Xem nội dung 1 file:**
  * **Cú pháp:** `cat [tên_file]`
  * **Ví dụ:** `cat /etc/hosts`
  * **Tại sao dùng?** Đọc luồng dữ liệu trên đĩa và đổ toàn bộ ra màn hình cùng một lúc. Phù hợp để xem nhanh các file cấu hình.
- **Xem dòng thứ `<n>` trong file:**
  * **Cú pháp:** `sed -n '[số_dòng]p' [tên_file]`
  * **Ví dụ (xem dòng 5):** `sed -n '5p' /etc/hosts`
  * **Tại sao lại kết hợp `sed`?** Bản chất `cat` không có bộ đếm dòng. Phải mượn `sed` để quét luồng văn bản, lệnh `-n` sẽ giấu tất cả các dòng, và `p` (print) chỉ in đúng dòng được chỉ định.
- **Ghi nhiều dòng vào 1 file bằng EOF:**
  * **Cú pháp:** ```bash
    cat <<EOF > [tên_file]
    [Nội_dung_dòng_1]
    [Nội_dung_dòng_2]
    EOF
    ```
  * **Ví dụ:**
    ```bash
    cat <<EOF > test.txt
    Day la dong 1
    Day la dong 2
    EOF
    ```
  * **Tại sao dùng EOF?** `<<EOF` (Here Document) báo cho hệ thống liên tục nhận dữ liệu đầu vào cho đến khi gặp chữ EOF, giúp ghi hàng loạt dòng mà không cần mở trình soạn thảo (vi, nano).

## 2. Tail / Head Command
- **Sự khác biệt giữa `tail` và `head`:**
  * **Cú pháp Head:** `head -n [số_dòng] [tên_file]` -> Lấy phần mào đầu của file.
  * **Cú pháp Tail:** `tail -n [số_dòng] [tên_file]` -> Nhảy xuống cuối file và đọc ngược lên, thường dùng để xem log lỗi mới nhất.
- **Sự khác biệt giữa `tail` và `tailf`:**
  * **Cú pháp Tail follow:** `tail -f [tên_file]` -> Liên tục mở file để đọc định kỳ, tốn tài nguyên I/O ổ cứng.
  * **Cú pháp Tailf:** `tailf [tên_file]` -> **Tại sao tối ưu hơn?** Nó dựa vào metadata của file. Chỉ khi hệ thống báo kích thước file tăng lên thì nó mới truy cập đĩa cứng, tiết kiệm tài nguyên Server.


# E. NHÓM 2: CHỈNH SỬA VÀ TẠO NỘI DUNG (EDIT)

## 1. Echo Command
- **Chèn thêm 1 dòng vào cuối file:**
  * **Cú pháp:** `echo "[Nội_dung]" >> [tên_file]`
  * **Ví dụ:** `echo "Dong moi" >> file.txt`
  * **Tại sao dùng `>>`?** Toán tử `>>` mở file ở chế độ **Append**. Nó đẩy con trỏ xuống cuối để ghi nối tiếp, bảo toàn 100% dữ liệu cũ.
- **Overwrite nội dung file:**
  * **Cú pháp:** `echo "[Nội_dung]" > [tên_file]`
  * **Ví dụ:** `echo "Xoa sach" > file.txt`
  * **Tại sao dùng `>`?** Toán tử `>` mở file ở chế độ **Truncate**. Ngay lập tức xóa sạch mọi dữ liệu đưa file về 0 byte rồi mới ghi nội dung mới.

## 2. Sed Command
- **Tìm và thay thế 1 chuỗi trong file:**
  * **Cú pháp:** `sed -i 's/[chuỗi_cũ]/[chuỗi_mới]/g' [tên_file]`
  * **Ví dụ:** `sed -i 's/port 22/port 2222/g' sshd_config`
  * **Tại sao dùng tham số này?** 
	+ `s` (substitute) là thay thế; 
	+ `g` (global) để thay thế tất cả các chữ trùng khớp trên cùng một dòng. 
	Lệnh giúp tự động hóa cấu hình mà không sợ gõ tay sai sót.


# F. NHÓM 3: QUẢN LÝ TỆP TIN / THƯ MỤC (MANAGE)

## 1. Find Command
- **Tìm file đuôi `.log`:**
  * **Cú pháp:** `find [đường_dẫn] -type f -name "*.[đuôi_file]"`
  * **Ví dụ:** `find /var/log -type f -name "*.log"`
  * **Tại sao dùng `-type f`?** Báo cho hệ thống chỉ tìm tập tin (file), bỏ qua thư mục.
- **Tìm folder tên `abc`:**
  * **Cú pháp:** `find [đường_dẫn] -type d -name "[tên_thư_mục]"`
  * **Ví dụ:** `find / -type d -name "abc"`
  * **Tại sao dùng `-type d`?** Ép hệ thống chỉ duyệt qua bảng cấu trúc thư mục (directory), tăng tốc độ tìm kiếm.
- **Tìm file tên `abc`:**
  * **Cú pháp:** `find [đường_dẫn] -type f -name "[tên_file]"`
  * **Ví dụ:** `find / -type f -name "abc"`
- **Tìm file `abc` và đặt quyền read only (444):**
  * **Cú pháp:** `find [đường_dẫn] -type f -name "[tên]" -exec chmod [quyền] {} \;`
  * **Ví dụ:** `find / -type f -name "abc" -exec chmod 444 {} \;`
  * **Tại sao làm vậy?** `-exec` lấy kết quả tìm được (nằm trong `{}`) nạp làm đầu vào cho lệnh `chmod`. Quyền `444` chặn ghi/thực thi, bảo vệ file quan trọng.

## 2. Cp Command 
- **Copy file:**
  * **Cú pháp:** `cp [file_nguồn] [file_đích]`
  * **Ví dụ:** `cp file1.txt file2.txt`
  * **Bản chất:** Tạo bản sao độc lập với Inode (địa chỉ dữ liệu) mới trên ổ cứng.
- **Copy folder:**
  * **Cú pháp:** `cp -r [thư_mục_nguồn]/ [thư_mục_đích]/`
  * **Ví dụ:** `cp -r /var/www/ /backup/`
  * **Tại sao cần `-r`?** Đệ quy (recursive) chỉ thị hệ thống đi sâu vào nhánh thư mục con để copy.

## 3. Mv Command (Di chuyển / Đổi tên)
- **Di chuyển và Đổi tên:**
  * **Cú pháp:** `mv [nguồn] [đích]`
  * **Ví dụ chuyển:** `mv test.txt /tmp/`
  * **Ví dụ đổi tên:** `mv ten_cu.txt ten_moi.txt`
  * **Tại sao dùng chung 1 lệnh?** `mv` không di dời dữ liệu vật lý trên ổ đĩa. Nó chỉ thay đổi con trỏ đường dẫn (path) trỏ đến khối dữ liệu đó.


# G. NHÓM 4: TRÍCH XUẤT VÀ PHÂN TÍCH VĂN BẢN (ANALYZE)

## 1. Sort Command (Sắp xếp)
- **Theo thứ tự tăng dần:**
  * **Cú pháp:** `sort [tên_file]`
  * **Bản chất:** Đọc mã ASCII của ký tự đầu tiên để xếp thứ tự.
- **Theo thứ tự giảm dần:**
  * **Cú pháp:** `sort -r [tên_file]`
  * **Tại sao dùng `-r`?** (reverse) đảo ngược kết quả, đưa dữ liệu lớn nhất (VD: IP tấn công) lên đầu.
- **Theo column:**
  * **Cú pháp:** `sort -k [số_cột] [tên_file]`
  * **Ví dụ:** `sort -k 2 file.txt`
  * **Tại sao dùng `-k`?** Lấy khoảng trắng làm vách ngăn và chỉ sắp xếp ưu tiên dựa trên dữ liệu ở cột chỉ định.

## 2. Uniq Command (Lọc trùng lặp)
- **Lọc các dòng lặp lại:**
  * **Cú pháp:** `sort [tên_file] | uniq`
  * **Tại sao phải `sort` trước?** `uniq` chỉ so sánh 2 dòng liền kề. Nếu các dòng giống nhau không được xếp cạnh nhau bằng `sort` trước, `uniq` sẽ bỏ sót.
- **Lọc và đếm số lượng dòng lặp lại:**
  * **Cú pháp:** `sort [tên_file] | uniq -c`
  * **Tại sao dùng `-c`?** (count) đính kèm số lượng ở đầu dòng, giúp phân tích tần suất.

## 3. Wc Command (Word Count)
- **Đếm số dòng:**
  * **Cú pháp:** `wc -l [tên_file]`
  * **Bản chất:** Đếm số lượng ký tự ngắt dòng (`\n`).
- **Đếm số ký tự:**
  * **Cú pháp:** `wc -m [tên_file]`

## 4. Cut Command (Cắt chuỗi)
- **Lấy ký tự thứ `<n>`:**
  * **Cú pháp:** `cut -c [vị_trí] [tên_file]`
  * **Ví dụ:** `cut -c 5 file.txt`
  * **Tại sao dùng `-c`?** Đóng vai trò như dao cắt dọc văn bản theo vị trí ký tự (character).
- **Lấy từ ký tự `<n>` trở về sau:**
  * **Cú pháp:** `cut -c [vị_trí]- [tên_file]`
  * **Ví dụ:** `cut -c 5- file.txt`
- **Lấy đến ký tự thứ `<n>`:**
  * **Cú pháp:** `cut -c -[vị_trí] [tên_file]`
  * **Ví dụ:** `cut -c -5 file.txt`


# H. NHÓM 5: BẢO MẬT VÀ PHÂN QUYỀN (SECURITY)

## 1. Chmod, Chown, Chattr Command
- **Phân quyền bằng số và chữ (Chmod):**
  * **Cú pháp số:** `chmod [quyền_bằng_số] [tên_file]`
  * **Ví dụ:** `chmod 755 file.sh` -> *Tại sao?* Dùng hệ nhị phân cộng lại (Read=4, Write=2, Execute=1) để định quyền cho User-Group-Other nhanh chóng.
  * **Cú pháp chữ:** `chmod [u/g/o][+/-][r/w/x] [tên_file]`
  * **Ví dụ:** `chmod u+x file.sh` -> *Tại sao?* An toàn khi chỉ muốn thêm/bớt một quyền cụ thể (VD: thêm quyền chạy cho User) mà không đụng chạm quyền khác.
- **Đổi owner user/group (Chown):**
  * **Cú pháp:** `chown [user]:[group] [tên_file_hoặc_thư_mục]`
  * **Ví dụ:** `chown root:admin test.txt`
  * **Tại sao cần đổi?** Giúp chuyển giao quyền xử lý file (VD: giao thư mục cho user `nginx` để tránh lỗi 403 Forbidden).
- **Set Immutable Attribute (Chattr):**
  * **Cú pháp:** `chattr +i [tên_file]`
  * **Tại sao lệnh này quan trọng?** Thay đổi cờ (flag) của File System thành "Bất biến". Chặn mọi thao tác Write/Delete từ tầng Kernel. Cực kỳ thiết yếu để ngăn mã độc thay đổi cấu hình.

# I. NHÓM 6: KIỂM TRA SỰ CỐ MẠNG (NETWORK TROUBLESHOOTING)

##1. Traceroute / Tracert Command
- **Thực hiện:**
  * **Cú pháp:** `traceroute [domain_hoặc_IP]` 
  * **Ví dụ:** `traceroute vietnix.vn`
- **Giải thích & Lý do:** Lệnh gửi các gói tin có thời gian sống (TTL) tăng dần. Qua mỗi Router, TTL hết hạn và Router đó phải phản hồi. Giúp kỹ thuật viên vẽ lộ trình mạng, biết kết nối đứt hay nghẽn ở nhà mạng (ISP) hay tại Data Center.

##2. Netstat Command (Quản lý Socket)
**Cú pháp tổng hợp thường dùng:** `netstat -tulnp`
- **Hiển thị các socket đang listen:** Cú pháp: `netstat -l` -> Chỉ lấy các cổng đang mở chờ kết nối tới.
- **Không resolve hostname:** Cú pháp: `netstat -n` -> Bỏ qua phân giải Hostname ra IP (rất chậm), in thẳng IP.
- **Không resolve portname:** Cú pháp: `netstat -n` -> Tương tự, in thẳng số Port thay vì tên dịch vụ (VD in 80 thay vì http).
- **Display process name/PID:** Cú pháp: `netstat -p` -> Gắn kết nối mạng với tiến trình cụ thể, giúp biết phần mềm nào đang chiếm cổng.
- **Chỉ hiển thị socket TCP:** Cú pháp: `netstat -t`
- **Chỉ hiển thị socket UDP:** Cú pháp: `netstat -u`

## 3. Dig Command (Kiểm tra DNS)
- **Kiểm tra record A, MX, NS:**
  * **Cú pháp:** `dig [tên_miền] [loại_record]`
  * **Ví dụ:**
    `dig vietnix.vn A` (IP gắn với domain)
    `dig vietnix.vn MX` (Mail Server)
    `dig vietnix.vn NS` (Máy chủ quản lý tên miền)
  * **Tại sao dùng Dig?** Truy vấn trực tiếp hệ thống DNS Server lấy dữ liệu thô chi tiết, chính xác hơn Ping.
- **Kiểm tra với custom DNS:**
  * **Cú pháp:** `dig @[IP_máy_chủ_DNS] [tên_miền] [loại_record]`
  * **Ví dụ:** `dig @8.8.8.8 vietnix.vn A`
  * **Tại sao?** Ép máy tính hỏi máy chủ độc lập (Google) thay vì DNS nhà mạng, dùng để đối chiếu xem nhà mạng có đang lưu cache cũ không.


# K. NHÓM 7: NÉN VÀ GIẢI NÉN TỆP TIN (ARCHIVE)

## 1. Tar / Zip / Unzip Command
- **Nén/giải nén `tar.gz`:**
  * **Cú pháp nén:** `tar -czvf [tên_file.tar.gz] [thư_mục_nguồn]/`
  * **Cú pháp giải nén:** `tar -xzvf [tên_file.tar.gz]`
  * **Tại sao dùng?** Gom file và giữ nguyên hoàn toàn hệ thống phân quyền Linux, chuẩn nén an toàn nhất cho Server.
- **Nén/giải nén `.zip`:**
  * **Cú pháp nén:** `zip -r [tên_file.zip] [thư_mục_nguồn]/` (Bắt buộc có `-r`).
  * **Cú pháp giải nén:** `unzip [tên_file.zip]`
  * **Tại sao dùng?** Chuẩn nén đa nền tảng, dùng khi xuất source code gửi cho khách hàng dùng Windows.
	
## NHÓM 8: QUẢN LÝ LƯU TRỮ VÀ Ổ CỨNG (STORAGE MANAGEMENT)

### 1. Df Command (Kiểm tra dung lượng đĩa)
- **Xem dung lượng disk:**
  * **Cú pháp:** `df -h`
  * **Ví dụ:** `df -h /var/log`
  * **Tại sao dùng `-h`?** `-h` (human-readable) giúp chuyển đổi các con số byte khổng lồ thành định dạng dễ đọc cho con người (MB, GB, TB). Nếu không có cờ này, kết quả toàn là số kilobyte rất khó nhìn.
- **Phân vùng `/` (Root Partition) là gì?**
  * Trong Linux, `/` là thư mục gốc (Root), là đỉnh cao nhất của cây thư mục. Khác với Windows chia ra ổ C, D, E độc lập; Linux gắn (mount) mọi phân vùng, mọi ổ cứng vật lý vào bên trong phân vùng `/` này. Nếu phân vùng `/` bị đầy 100%, toàn bộ hệ điều hành và các dịch vụ (web, database) sẽ ngừng hoạt động ngay lập tức.

### 2. Mount / Umount Command (Gắn và Tháo ổ cứng)
*Bối cảnh: Bạn vừa mua thêm 1 ổ cứng 5GB và cắm vào Server. Hệ thống nhận diện nó là `/dev/sdb`.*
- **Kiểm tra số lượng ổ cứng đang cắm vào máy:**
  * **Cú pháp:** `lsblk` hoặc `fdisk -l`
  * **Tại sao?** Lệnh này liệt kê danh sách các khối thiết bị (block devices). Phải dùng lệnh này để lấy được tên chính xác của ổ cứng mới (VD: `sdb`, `nvme0n1`) trước khi thao tác.
- **Mount vào `/mnt/test` (Gắn ổ cứng):**
  * **Cú pháp:** `mount [ổ_cứng_vật_lý] [thư_mục_trống]`
  * **Ví dụ:** `mount /dev/sdb /mnt/test`
  * **Tại sao phải Mount?** Linux không tự động hiện ổ đĩa mới như Windows. Bạn bắt buộc phải tạo một thư mục trống (gọi là Mount Point, ở đây là `/mnt/test`), sau đó "móc" ổ cứng vật lý `/dev/sdb` vào thư mục đó thì mới có thể đọc/ghi dữ liệu.
- **Umount `/mnt/test` (Tháo ổ cứng an toàn):**
  * **Cú pháp:** `umount [thư_mục_đang_mount]`
  * **Ví dụ:** `umount /mnt/test`
  * **Tại sao phải Umount?** Để ép hệ thống ghi toàn bộ dữ liệu còn đang lưu tạm trong RAM xuống hẳn ổ cứng, ngắt kết nối an toàn trước khi bạn rút ổ cứng ra vật lý hoặc format nó.


## NHÓM 9: GIÁM SÁT TÀI NGUYÊN & TIẾN TRÌNH (MONITORING)

### 1. Top Command (Giám sát CPU và Hệ thống Real-time)
- **Kiểm tra tài nguyên CPU:**
  * **Cú pháp:** `top` (Nhấn phím `q` để thoát).
  * **Tại sao dùng?** Cung cấp một bảng điều khiển trực tiếp (real-time) về toàn bộ tình trạng của Server, giống như Task Manager trên Windows.
- **Giải thích các thông số sống còn của Top:**
  * **load average (1, 5, 15 phút):** Con số thể hiện độ chịu tải của hệ thống. Nếu Server có 4 nhân CPU mà load average > 4.0, nghĩa là Server đang quá tải (overload), có tiến trình phải xếp hàng chờ xử lý.
  * **%Cpu(s):**
    * `us` (user): % CPU đang xử lý tác vụ của người dùng (như chạy code PHP, Nginx).
    * `sy` (system): % CPU đang xử lý tác vụ cốt lõi của hệ điều hành.
    * `id` (idle): % CPU đang rảnh rỗi. Số này càng cao càng tốt.
    * `wa` (IO-wait): % CPU đang phải "ngồi chơi xơi nước" để chờ ổ cứng đọc/ghi dữ liệu. Nếu số này cao, ổ cứng của bạn đang bị nghẽn (quá chậm hoặc lỗi).

### 2. Free Command (Giám sát RAM)
- **Xem dung lượng RAM:**
  * **Cú pháp:** `free -m` (hiển thị theo Megabyte) hoặc `free -h` (Human-readable).
- **Giải thích các thông số về RAM:**
  * **total:** Tổng dung lượng RAM vật lý của máy.
  * **used:** RAM thực sự đang bị các phần mềm chiếm dụng.
  * **buff/cache:** Đặc sản của Linux! Linux có cơ chế tự động lấy phần RAM rảnh rỗi để làm bộ nhớ đệm (cache) cho ổ cứng giúp web chạy nhanh hơn. Nếu ứng dụng cần RAM, Linux sẽ tự nhả phần cache này ra.
  * **available (Quan trọng nhất):** Dung lượng RAM thực tế SẴN SÀNG để cấp cho một ứng dụng mới. Khi kiểm tra xem Server có thiếu RAM không, Technical Staff nhìn vào cột `available` chứ không nhìn cột `free`.

### 3. Ps Command (Quản lý Tiến trình)
- **Show tiến trình đang chạy:**
  * **Cú pháp:** `ps [tham_số]`
  * **Ví dụ thực chiến:** `ps aux`
  * **Tại sao dùng `aux`?** `a` hiển thị tiến trình của tất cả người dùng, `u` hiển thị định dạng có tên user, `x` hiển thị cả các tiến trình chạy ngầm không có giao diện (daemon). Giúp bạn nhìn thấy toàn bộ bức tranh của hệ thống.
- **Kill tiến trình (Tiêu diệt):**
  * **Cú pháp:** `kill -9 [PID]` (PID là số Process ID lấy từ lệnh `ps` hoặc `top`).
  * **Ví dụ:** `kill -9 1234`
  * **Tại sao dùng `-9`?** Đây là cờ `SIGKILL` (Tín hiệu hủy diệt). Lệnh `kill` thông thường sẽ "xin phép" phần mềm tự đóng, nếu phần mềm bị treo nó sẽ phớt lờ. Cờ `-9` ra lệnh cho hệ điều hành cưỡng chế "giết" phần mềm ngay lập tức không cần chờ đợi.


## NHÓM 10: LIÊN KẾT VÀ HIỂN THỊ TẬP TIN (FILE LISTING & LINKING)

### 1. Ls Command (Liệt kê dữ liệu)
- **Liệt kê file/thư mục cơ bản:**
  * **Cú pháp:** `ls [thư_mục]`
- **Liệt kê file/thư mục và thuộc tính chi tiết:**
  * **Cú pháp:** `ls -l`
  * **Tại sao dùng `-l`?** (Long format) Nó vẽ ra một bảng chi tiết hiển thị quyền hạn (rwx), chủ sở hữu, dung lượng và ngày giờ chỉnh sửa cuối cùng của file. Rất quan trọng để check lỗi phân quyền.
- **Show file ẩn:**
  * **Cú pháp:** `ls -a` (Thường kết hợp thành `ls -la`).
  * **Tại sao dùng `-a`?** (All) Trong Linux, bất kỳ file hay thư mục nào có tên bắt đầu bằng dấu chấm (VD: `.htaccess`, `.ssh`) đều bị ẩn đi. Phải có cờ `-a` mới nhìn thấy chúng.

### 2. Symbolic Links & Hard Links Command (Tạo liên kết)
*Lệnh sử dụng: `ln` (Link)*
- **Định nghĩa Symbolic Link (Soft Link):**
  * Hoạt động y hệt như chức năng "Shortcut" trên Windows. Nó là một file nhỏ trỏ *đường dẫn* đến file gốc.
  * *Hệ quả:* Nếu bạn xóa file gốc, Symlink sẽ trở thành một liên kết chết (Dangling link) vô tác dụng.
- **Định nghĩa Hard Link:**
  * Trỏ trực tiếp vào `Inode` (vị trí vật lý chứa dữ liệu trên ổ cứng), tạo ra một "ngôi nhà có 2 cánh cửa". Hai file này chia sẻ chung dữ liệu.
  * *Hệ quả:* Nếu xóa file gốc, dữ liệu vẫn còn nguyên và vẫn có thể truy cập thông qua file Hard Link.
- **Ví dụ về Symlink và Hardlink:**
  * **Tạo Symlink:** `ln -s /var/www/html/ /home/user/shortcut_web`
    *(Tại sao dùng `-s`? Chữ `s` đại diện cho symbolic/soft, bắt buộc phải có để hệ thống không tạo nhầm thành hard link).*
  * **Tạo Hardlink:** `ln data_goc.txt ban_sao_hardlink.txt`