
# INTERNET FUNDAMENTALS (NỀN TẢNG INTERNET)

## I. KIẾN THỨC VỀ SSL/TLS (BẢO MẬT WEBSITE)

### 1. SSL là gì?
- **Định nghĩa:** SSL (Secure Sockets Layer) là tiêu chuẩn an ninh công nghệ toàn cầu tạo ra một liên kết được mã hóa giữa máy chủ web (Web Server) và trình duyệt (Browser). 
- **Tác dụng thực tế:** Đảm bảo mọi dữ liệu truyền đi (mật khẩu, thẻ tín dụng) không bị hacker đánh cắp. Khi cài SSL, website sẽ chuyển từ `http://` sang `https://` và có biểu tượng ổ khóa an toàn.

### 2. Có bao nhiêu cách xác thực SSL?
Có 3 mức độ xác thực chứng chỉ SSL từ thấp đến cao:
1. **DV (Domain Validation):** Xác thực quyền sở hữu tên miền. Chỉ cần chứng minh bạn quản lý tên miền (qua Email, upload file text hoặc cấu hình DNS). Cấp phát trong vài phút, phù hợp web cá nhân.
2. **OV (Organization Validation):** Xác thực tổ chức. CA (Certificate Authority) sẽ kiểm tra giấy phép kinh doanh của công ty. Mất 1-3 ngày, độ tin cậy cao hơn.
3. **EV (Extended Validation):** Xác thực mở rộng. Mức độ cao nhất, quy trình kiểm tra công ty rất khắt khe. Trước đây thanh địa chỉ sẽ hiện màu xanh lá cây chứa tên công ty (dành cho Ngân hàng, web tài chính lớn).

### 3. CSR file dùng để làm gì?
- **Định nghĩa:** CSR (Certificate Signing Request) là một "Đơn đăng ký" được tạo ra trên Server để gửi cho tổ chức cấp phát SSL (CA).
- **Chức năng:** Nó chứa thông tin của website (Tên miền, Tên công ty, Quốc gia) và chứa **Public Key** của bạn. Tổ chức CA sẽ dùng file CSR này để tạo ra chứng chỉ số (CRT) cho bạn.

### 4. Gen file CSR và request SSL bằng OpenSSL
*Yêu cầu: Tạo CSR cho subdomain `tech.training.vietnix.tech`*

- **Cú pháp thực hiện trên Terminal/Linux:**
  ```bash
  openssl req -new -newkey rsa:2048 -nodes -keyout tech.training.vietnix.tech.key -out tech.training.vietnix.tech.csr
  ```
- **Giải thích tham số thực chiến:**
  - `req -new`: Yêu cầu tạo mới một CSR.
  - `-newkey rsa:2048`: Đồng thời tạo luôn một Private Key mới bằng thuật toán RSA độ dài 2048-bit (chuẩn an toàn hiện nay).
  - `-nodes`: (No DES) Bỏ qua bước đặt mật khẩu cho file Key để Web Server (như Nginx/Apache) có thể tự động đọc Key khi khởi động lại máy chủ mà không cần người gõ pass.
  - `-keyout` và `-out`: Tên file Private Key và CSR sẽ được tạo ra.

### 5. Pem, Private Key, PFX là gì?
- **Pem file là gì?** (Privacy Enhanced Mail). Đây là một định dạng file chứa dữ liệu mã hóa Base64. Trong thực tế, đuôi `.pem` thường được dùng để chứa "một gói hỗn hợp" (bao gồm cả CRT, CA Bundle và đôi khi cả Private Key) để tiện cho việc cài đặt trên một số hệ thống.
- **Private Key SSL là gì?** Đây là "Chìa khóa vạn năng" nằm trên Web Server, dùng để giải mã các dữ liệu mà trình duyệt của người dùng đã mã hóa. **Lưu ý tối quan trọng:** Private key tuyệt đối không được để lộ, không được gửi cho bất kỳ ai. Nếu mất Private Key, chứng chỉ SSL (CRT) tương ứng sẽ trở nên vô dụng và phải làm lại từ đầu.
- **PFX file là gì?** (PKCS#12). Khác với Linux dùng file rời (.crt và .key), hệ thống **Windows Server (IIS)** yêu cầu tất cả chứng chỉ (CRT), CA Bundle và Private Key phải được đóng gói chung vào 1 file duy nhất có đuôi `.pfx` và được bảo vệ bằng mật khẩu.

- **Cách chuyển từ CRT sang PFX (Bằng OpenSSL):**
  ```bash
  openssl pkcs12 -export -out certificate.pfx -inkey private.key -in certificate.crt
  ```


## II. KIẾN THỨC VỀ DOMAIN (TÊN MIỀN)

### 1. Domain là gì?
Domain (Tên miền) giống như "địa chỉ nhà" của bạn trên Internet (Ví dụ: `vietnix.vn`). Nó thay thế cho những dãy địa chỉ IP máy chủ khô khan (Ví dụ: `103.200.23.123`) giúp con người dễ nhớ và truy cập.

### 2. Các trạng thái (Vòng đời) của Domain
Khi hỗ trợ khách hàng, cần check WHOIS để xem trạng thái domain đang nằm ở đâu:
1. **Available (Tự do):** Tên miền chưa ai đăng ký, có thể mua ngay.
2. **Active (Đang hoạt động):** Tên miền đã được mua và đang dùng bình thường.
3. **Expired / Grace Period (Hết hạn / Chờ gia hạn):** Từ 0-30 ngày sau khi hết hạn. Web ngừng hoạt động nhưng chủ cũ vẫn có thể gia hạn với giá gốc.
4. **Redemption (Chờ chuộc):** Từ 30-45 ngày tiếp theo. Muốn lấy lại tên miền phải trả phí chuộc cực cao (gấp nhiều lần giá gốc).
5. **Pending Delete (Chờ xóa):** Khoảng 5 ngày. Tên miền đã bị thu hồi, không ai có thể can thiệp. Chờ nó quay về trạng thái Available để đăng ký lại.

### 3. Subdomain là gì?
- Là tên miền phụ, một nhánh mở rộng được tạo ra từ tên miền chính. 
- *Ví dụ:* Tên miền chính là `vietnix.vn`. Ta có thể tạo ra các subdomain như `portal.vietnix.vn` (để quản lý dịch vụ) hoặc `tech.training.vietnix.tech` (để training). Subdomain hoạt động như một website hoàn toàn độc lập và không tốn phí đăng ký.

### 4. Virtual Hosts là gì?
- Đây là một kỹ thuật cấu hình trên Web Server (Apache hoặc cấu hình Server Block trên Nginx).
- **Tác dụng thực chiến:** Cho phép bạn chạy **nhiều website (nhiều domain khác nhau)** trên cùng **1 địa chỉ IP** và cùng 1 máy chủ vật lý. Web Server sẽ nhìn vào tên miền khách truy cập gõ để trả về đúng thư mục chứa mã nguồn của web đó.


## III. MAIL SERVER 

### 1. Tìm hiểu MX Record
- **MX (Mail Exchanger):** Là bản ghi DNS đặc biệt dùng để định tuyến luồng Email. Nó chỉ cho các hệ thống khác biết: "Để gửi mail cho tên miền @vietnix.vn, hãy giao nó cho máy chủ có địa chỉ IP này".
- Nếu web chết, người dùng không xem được web. Nhưng nếu MX Record lỗi, toàn bộ email gửi đến công ty sẽ bị thất lạc.

### 2. Bộ 3 bản ghi chống Spam (DKIM, SPF, PTR)
Khi khách hàng báo: *"Em ơi sao mail anh gửi toàn vào hộp thư rác (Spam) của đối tác?"*, Technical Staff phải check ngay 3 bản ghi này:
- **SPF (Sender Policy Framework):** Bản ghi dạng TXT. Nó khai báo công khai cho thế giới biết những IP/Server nào được phép nhân danh tên miền của bạn để gửi mail. Tránh bị kẻ xấu giả mạo email công ty.
- **DKIM (DomainKeys Identified Mail):** Bản ghi dạng TXT chứa Public Key. Máy chủ mail gửi sẽ ký một chữ ký điện tử ngầm vào từng bức thư. Máy chủ nhận sẽ dùng DKIM để xác minh thư này chưa bị chỉnh sửa nội dung trên đường truyền.
- **PTR (Pointer Record - Reverse DNS):** Ngược lại với DNS thông thường (từ Domain ra IP), PTR phân giải từ IP ra ngược lại Domain. Máy chủ nhận mail (như Gmail) rất khó tính, nó sẽ check ngược IP của máy chủ gửi xem có đúng là của tên miền đó không. Cấu hình PTR không thực hiện trên DNS tên miền, mà phải làm trên quản lý IP của Data Center.


## IV. DNS (DOMAIN NAME SYSTEM)

### 1. DNS là gì?
DNS (Hệ thống phân giải tên miền) được ví như **Danh bạ điện thoại của Internet**. Con người nhớ tên (Domain), máy tính nhớ số (IP). DNS làm nhiệm vụ "phiên dịch" qua lại giữa tên miền và IP để trình duyệt có thể tải được web.

### 2. Các loại bản ghi (Record) DNS cơ bản
- **A Record:** Trỏ tên miền về một địa chỉ IPv4 (Ví dụ: 103.200.23.123).
- **AAAA Record:** Trỏ tên miền về một địa chỉ IPv6.
- **CNAME Record:** (Alias) Trỏ tên miền này sang một tên miền khác. (Ví dụ: `www.vietnix.vn` trỏ về `vietnix.vn`). Không được dùng CNAME cho tên miền trọc (root domain).
- **MX Record:** Trỏ về máy chủ nhận Email.
- **TXT Record:** Chứa các đoạn văn bản, thường dùng để cấu hình SPF, DKIM hoặc xác minh sở hữu tên miền với Google.
- **NS Record:** (Name Server) Chỉ định máy chủ nào đang nắm quyền quản lý các bản ghi DNS của tên miền này.

### 3. Nguyên tắc làm việc của DNS
- **Phân tán & Thứ bậc:** Không có một máy chủ DNS nào chứa toàn bộ danh bạ thế giới. Nó được chia thành nhiều cấp bậc (Root -> TLD -> Authoritative).
- **Cơ chế Caching (Lưu tạm):** Để web truy cập nhanh, kết quả DNS sẽ được lưu tạm ở máy tính người dùng và các nhà mạng (VNPT, FPT). Đây là lý do tại sao khi đổi IP web, phải mất 1 thời gian (từ vài phút đến 24h) để tất cả mạng cập nhật kết quả mới.

### 4. Quá trình phân giải địa chỉ DNS (Cách lấy IP)
*Khi người dùng gõ `vietnix.vn` vào trình duyệt, một cuộc hành trình cực nhanh diễn ra qua các bước:*

1. **Browser & OS Cache:** Trình duyệt tìm trong bộ nhớ tạm của nó và file `hosts` của máy tính. Nếu không có, nó sẽ hỏi máy chủ DNS của nhà mạng (ISP Resolver).
2. **DNS Resolver:** Máy chủ nhà mạng đóng vai "nhân viên tìm kiếm". Nó hỏi thẳng lên cấp cao nhất là **Root Nameserver**.
3. **Root Nameserver:** Root không biết IP của vietnix.vn, nhưng nó biết ai quản lý đuôi `.vn`, nên nó điều hướng nhân viên tìm kiếm đến máy chủ **TLD (Top-Level Domain) `.vn`**.
4. **TLD Nameserver:** Máy chủ TLD `.vn` nói: "Tôi không biết IP, nhưng tôi biết Name Server đang quản lý tên miền vietnix.vn là ai (Ví dụ ns1.vietnix.vn)".
5. **Authoritative Nameserver:** Nhân viên tìm kiếm chạy đến Name Server của Vietnix. Name Server này lục trong bảng record và trả về chính xác IP `103.200.23.x`.
6. **Trả kết quả:** Nhân viên (DNS Resolver) lưu kết quả này vào cache rồi báo lại cho Trình duyệt. Trình duyệt bắt đầu kết nối tới IP đó để tải website!
