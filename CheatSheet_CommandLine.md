# LINUX QUICK-LOOK: TÌNH HUỐNG & GIẢI PHÁP NẾU GẶP

## 1. XỬ LÝ SỰ CỐ WEBSITE & PHÂN QUYỀN
| Nếu gặp tình huống... | Hãy dùng lệnh | Giải thích |
| :--- | :--- | :--- |
| **Web báo lỗi 403 Forbidden** | `chown -R nginx:nginx /var/www/html` | Đổi chủ sở hữu thư mục web về đúng user chạy dịch vụ. |
| **Cần fix quyền file/folder chuẩn** | `find . -type d -exec chmod 755 {} \;` | Folder để 755, File để 644 là an toàn nhất. |
| **Web bị chèn mã độc liên tục** | `chattr +i index.php` | Khóa chết file, không cho bất kỳ ai (kể cả root) sửa/xóa. |
| **Cần tìm file cấu hình bị thất lạc** | `find / -name "wp-config.php"` | Tìm kiếm file từ thư mục gốc. |

## 2. KIỂM TRA MẠNG & KẾT NỐI
| Nếu gặp tình huống... | Hãy dùng lệnh | Giải thích |
| :--- | :--- | :--- |
| **Cần biết Port 80 có ai dùng chưa** | `netstat -tulnp | grep :80` | Kiểm tra xem Nginx hay Apache đang chiếm port. |
| **Khách than mạng chậm/lag** | `traceroute [IP_Khách]` | Xem gói tin bị nghẽn ở nhà mạng nào (VNPT/Viettel...). |
| **Check domain đã trỏ đúng IP chưa** | `dig @8.8.8.8 domain.com A` | Hỏi thẳng Google để tránh bị cache DNS nhà mạng. |
| **Copy data giữa 2 Server cực nhanh** | `rsync -avz /source/ root@IP:/dest/` | Chỉ copy những gì thay đổi, tự nối lại nếu rớt mạng. |

## 3. GIÁM SÁT TÌNH TRẠNG SERVER
| Nếu gặp tình huống... | Hãy dùng lệnh | Giải thích |
| :--- | :--- | :--- |
| **Server bỗng dưng chạy chậm** | `top` | Nhìn `load average` > số CPU là đang quá tải. |
| **Web báo lỗi "No space left"** | `df -h` | Kiểm tra xem ổ cứng phân vùng `/` có bị đầy 100% không. |
| **Cần biết RAM còn trống thực sự** | `free -m` | Nhìn vào cột **available** (đừng nhìn cột free). |
| **Tắt một tiến trình đang bị treo** | `kill -9 [PID]` | Kill tiến trình ngay lập tức. |

## 4. THAO TÁC FILE NHANH
| Nếu gặp tình huống... | Hãy dùng lệnh | Giải thích |
| :--- | :--- | :--- |
| **Theo dõi log lỗi đang nhảy** | `tail -f /var/log/nginx/error.log` | Xem trực tiếp các lỗi khách đang gặp phải. |
| **Xóa sạch nội dung file log cũ** | `echo "" > access.log` | Làm trống file nhanh mà không cần xóa rồi tạo lại. |
| **Tìm và đổi hàng loạt Port trong file**| `sed -i 's/22/2222/g' config.conf` | Đổi toàn bộ số 22 thành 2222 . |
| **Giải nén file backup khách gửi** | `tar -xzvf backup.tar.gz` | Giải nén chuẩn Linux thường gặp. |
