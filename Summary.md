

#  LIÊN KẾT KIẾN THỨC BUỔI TRAINING VÀO THỰC TẾ

**Phạm vi kiến thức:** Linux Command Line & Internet Fundamentals


## BỐI CẢNH THỰC CHIẾN (TICKET CẤP CỨU TỪ KHÁCH HÀNG)

**Nội dung Ticket:** *"Chào kỹ thuật Vietnix, website bán hàng `store.doanhnghiep.com` của bên anh đang sập hoàn toàn (trắng trang). Nhân viên dev mới của anh lỡ tay cấu hình sai gì đó. Trước khi sập, web hay báo lỗi 403, và email gửi từ hệ thống toàn bị rớt vào hộp Spam của khách. Anh vừa nạp tiền mua thêm 1 ổ cứng 20GB cắm vào Server rồi mà sao hệ thống vẫn báo đầy? Xử lý gấp giúp anh!"*

**Nhận định ban đầu:**
Lỗi: Tầng Mạng (DNS) -> Tầng Lưu trữ (Ổ cứng) -> Tầng Dịch vụ (Web/SSL) -> Tầng Giao tiếp (Mail). 

QUY TRÌNH ĐỀ XUẤT:

## BƯỚC 1: CHẨN ĐOÁN TẦNG MẠNG VÀ PHÂN GIẢI (DNS & NETWORK)
*Mục tiêu: Đảm bảo tên miền khách hàng thực sự đang trỏ về máy chủ của Vietnix chứ không phải máy chủ cũ, và mạng không bị nghẽn.*

1. **Kiểm tra phân giải tên miền (Tránh Cache nhà mạng):**
   ```bash
   dig @8.8.8.8 store.doanhnghiep.com A
   ```
   * *Ý nghĩa:* Ép máy tính hỏi Google (8.8.8.8) xem tên miền này đang trỏ IP nào. Tránh tình trạng modem nhà khách hàng bị lưu cache cũ.
   * *Kết quả:* Xác nhận đã trỏ đúng IP máy chủ Vietnix (VD: 103.200.23.123).

2. **Kiểm tra trạng thái sống/chết của Server:**
   ```bash
   ping -c 4 103.200.23.123
   ```
   * *Ý nghĩa:* Gửi 4 gói tin ICMP. Nếu nhận đủ 4 phản hồi -> Mạng thông suốt, Server vẫn đang chạy ngầm, rớt web là do dịch vụ bên trong.


## BƯỚC 2: KIỂM TRA HỆ THỐNG VÀ XỬ LÝ LƯU TRỮ (SYSTEM & STORAGE)
*Mục tiêu: Xử lý vấn đề khách kêu "đã mua ổ cứng mà vẫn báo đầy". Server sập trắng trang thường do hết RAM hoặc đầy ổ cứng.*

1. **SSH vào Server và kiểm tra tài nguyên:**
   ```bash
   ssh root@103.200.23.123
   df -h
   ```
   * *Kết quả:* Lệnh `df -h` báo phân vùng gốc `/` đang `100% Use`. Đây là nguyên nhân gốc làm sập Web (Database không còn chỗ để ghi dữ liệu).

2. **Kiểm tra ổ cứng mới mua:**
   ```bash
   lsblk
   ```
   * *Ý nghĩa:* Liệt kê các khối ổ cứng vật lý. Phát hiện có một ổ `sdb` (20GB) đang cắm vào nhưng chưa được sử dụng. Khách hàng lầm tưởng mua ổ cứng là Server tự nhận như Windows.

3. **Gắn (Mount) ổ cứng mới để giải cứu hệ thống:**
   ```bash
   mkdir /mnt/data_backup
   mount /dev/sdb /mnt/data_backup
   ```
   * *Hành động:* Chuyển bớt các file log cũ, dung lượng lớn sang ổ cứng mới (`/mnt/data_backup`) bằng lệnh `mv`, trả lại không gian cho phân vùng gốc `/`.


## BƯỚC 3: GIẢI QUYẾT LỖI DỊCH VỤ WEB VÀ BẢO MẬT (WEB & SSL)
*Mục tiêu: Khắc phục lỗi 403 Forbidden và khôi phục Website.*

1. **Truy tìm nguyên nhân lỗi 403 qua Log:**
   ```bash
   tail -n 50 /var/log/nginx/error.log
   ```
   * *Kết quả:* Báo lỗi *Permission Denied*. (Dev của khách đã tải source code bằng quyền root nên Nginx không có quyền đọc).

2. **Cấp lại quyền sở hữu (Phân quyền):**
   ```bash
   chown -R nginx:nginx /var/www/store/
   find /var/www/store/ -type d -exec chmod 755 {} \;
   find /var/www/store/ -type f -exec chmod 644 {} \;
   ```
   * *Ý nghĩa:* Trả lại quyền sở hữu thư mục web cho `nginx`. Đảm bảo chuẩn bảo mật: Thư mục 755 (Đọc/Chạy), File 644 (Chỉ đọc).

3. **(Bổ sung) SSL/HTTPS bị mất do Dev làm hỏng file:**
   * Kỹ thuật viên hỗ trợ khách tạo lại file yêu cầu chứng chỉ (CSR) mới ngay trên Terminal:
   ```bash
   openssl req -new -newkey rsa:2048 -nodes -keyout store.key -out store.csr
   ```
   * *Ý nghĩa:* Tạo ngay một cặp Key và CSR (không pass) gửi cho hãng CA để cấp lại chứng chỉ CRT mới.


## BƯỚC 4: GIẢI QUYẾT LỖI EMAIL VÀO SPAM (MAIL SERVER & DNS)
*Mục tiêu: Đảm bảo độ tin cậy của Mail Server bằng các bản ghi chống Spam.*

1. **Kiểm tra luồng Mail đang bị kẹt:**
   ```bash
   tail -f /var/log/maillog
   ```
   * *Ý nghĩa:* Theo dõi realtime. Phát hiện Server đích (như Gmail) từ chối nhận thư do thiếu chứng thực tên miền.

2. **Kiểm tra các bản ghi định danh (DNS Record):**
   ```bash
   dig store.doanhnghiep.com MX
   dig store.doanhnghiep.com TXT
   ```
   * *Phân tích:* Khách hàng đã có bản ghi `MX` (để nhận mail), nhưng quên cấu hình bản ghi `TXT` (chứa SPF và DKIM). 
   * *Khắc phục:* Hướng dẫn khách hàng tạo bản ghi TXT khai báo SPF (cho phép IP của Vietnix gửi mail) trên trang quản lý tên miền.

