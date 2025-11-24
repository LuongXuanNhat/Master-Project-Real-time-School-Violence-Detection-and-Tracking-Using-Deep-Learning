📦 Những gì bạn cần chuẩn bị:
✅ Camera DS-2CD1023G0-IUF (đã có)
✅ Router WiFi ở nhà/văn phòng
✅ Dây mạng (cáp LAN) - thường đi kèm camera, nếu không có mua thêm (~10-20k/mét)
✅ Laptop có thể kết nối WiFi
✅ Điện thoại để cài app (tuỳ chọn)

🎯 Mục tiêu: Xem camera từ xa qua Internet
Hiểu đơn giản:
Camera → cắm dây mạng vào → Router WiFi → Internet → Laptop bạn ở bất kỳ đâu

📝 HƯỚNG DẪN CHI TIẾT (3 BƯỚC CHÍNH)
BƯỚC 1: Lắp đặt và kết nối camera 🔧
1.1. Lắp camera:

Gắn camera ở vị trí cần giám sát
Cắm nguồn adapter 12V vào camera (nếu không dùng PoE)

1.2. Kết nối dây mạng:
Camera [cổng RJ45] ←--[dây mạng LAN]--→ Router [cổng LAN]

Lấy dây mạng, một đầu cắm vào camera
Đầu kia cắm vào cổng LAN màu vàng của router WiFi

1.3. Kiểm tra đèn:

Đèn trên camera sáng → OK ✅
Đèn trên cổng router nhấp nháy → đã kết nối ✅


BƯỚC 2: Cài đặt camera lần đầu 💻
2.1. Tải phần mềm SADP (tìm camera):
Trên laptop:

Vào: https://www.hikvision.com/en/support/tools/
Tìm "SADP Tool" → Download
Cài đặt (Next → Next → Finish)

2.2. Tìm camera trên mạng:

Mở SADP Tool (icon màu xanh)
Đợi 10-20 giây, camera sẽ hiện ra danh sách
Bạn sẽ thấy:

   Device Name: DS-2CD1023G0-IUF
   IP Address: 192.168.1.64 (hoặc số khác)
   Status: Inactive (lần đầu)
2.3. Kích hoạt camera (lần đầu):

Click vào camera trong danh sách
Bên phải màn hình:

Create Password: tạo mật khẩu (ví dụ: Admin123456)
Confirm Password: nhập lại
Click Activate


Đợi 30 giây → Status chuyển thành Active ✅

2.4. Thay đổi IP tĩnh (quan trọng):
Để camera không đổi IP mỗi lần khởi động:

Click vào camera đã Active
Bên phải:

IP Address: đổi thành 192.168.1.100 (hoặc số cuối từ 100-200)
Subnet Mask: 255.255.255.0
Gateway: 192.168.1.1 (IP router của bạn)
Nhập password vừa tạo
Click Modify


Camera sẽ khởi động lại → xong ✅


BƯỚC 3: Xem camera từ xa 🌐
Có 2 cách dễ nhất:

CÁCH 1: Dùng App Hik-Connect (DỄ NHẤT) ⭐ Khuyến nghị
Trên điện thoại:

Tải app:

iPhone: App Store → tìm "Hik-Connect"
Android: CH Play → tìm "Hik-Connect"


Đăng ký tài khoản:

Mở app → Sign Up
Nhập email, tạo mật khẩu
Xác nhận email


Thêm camera:

Click dấu "+" góc trên
Chọn "Scan QR Code"
Quét mã QR trên thân camera hoặc hộp
Hoặc chọn "Manual Adding" → nhập:

Device Serial No.: (số serial trên camera, ví dụ: DS-2CD1023G0-IUF12345678)
Verification Code: (mã 6 chữ cái in trên camera)


Nhập password camera (đã tạo ở bước 2.3)
Click Add


Bật dịch vụ đám mây Hik-Connect:

Vào trình duyệt máy tính: http://192.168.1.100
Login: admin / mật khẩu đã tạo
Vào: Configuration → Network → Platform Access
Bật "Enable Hik-Connect"
Verification Code: nhập mã trên camera
Click Save


Xem từ xa:

Mở app Hik-Connect
Click vào camera → xem trực tiếp! 🎉
Hoạt động ở bất kỳ đâu (4G/5G/WiFi khác)




CÁCH 2: Truy cập trực tiếp qua DDNS (Nâng cao hơn)
Nếu muốn xem qua trình duyệt laptop hoặc code Python:
Bước 2.1: Tạo tài khoản DDNS miễn phí

Vào: https://www.noip.com
Sign Up → Free Account
Tạo hostname (tên miền miễn phí):

Ví dụ: mycamera123.ddns.net


Note lại username, password, hostname

Bước 2.2: Cấu hình DDNS trên camera

Vào http://192.168.1.100 (IP camera)
Login: admin / password
Vào: Configuration → Network → Advanced Settings → DDNS
Điền:

DDNS Type: NO-IP
Server Address: dynupdate.no-ip.com
Domain Name: mycamera123.ddns.net
Username: tài khoản no-ip
Password: mật khẩu no-ip
Click Save



Bước 2.3: Port Forwarding trên Router
Lưu ý: Mỗi router khác nhau, mình hướng dẫn chung:

Vào trang quản trị router:

Mở trình duyệt: http://192.168.1.1
Login (thường in sau lưng router)
Ví dụ: admin/admin hoặc admin/1234


Tìm mục Port Forwarding:

Có thể tên: Virtual Server / NAT / Port Forwarding
Thường ở tab "Advanced" hoặc "Firewall"


Thêm 2 rules:
Rule 1 - HTTP:

Service Name: Camera_HTTP
External Port: 8080
Internal Port: 80
Internal IP: 192.168.1.100 (IP camera)
Protocol: TCP
Click Save/Apply

Rule 2 - RTSP:

Service Name: Camera_RTSP
External Port: 554
Internal Port: 554
Internal IP: 192.168.1.100
Protocol: TCP
Click Save/Apply


Lưu lại IP Public của bạn:

Vào: https://whatismyipaddress.com
Note IP (ví dụ: 123.45.67.89)



Bước 2.4: Truy cập từ xa
Qua trình duyệt:
http://mycamera123.ddns.net:8080
hoặc
http://123.45.67.89:8080
Qua Python (code AI):
pythonimport cv2

# Cách 1: Dùng DDNS
rtsp_url = "rtsp://admin:Admin123456@mycamera123.ddns.net:554/Streaming/Channels/101"

# Cách 2: Dùng IP Public
rtsp_url = "rtsp://admin:Admin123456@123.45.67.89:554/Streaming/Channels/101"

cap = cv2.VideoCapture(rtsp_url)

while True:
    ret, frame = cap.read()
    if ret:
        cv2.imshow('Camera', frame)
    
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()

🎯 TÓM TẮT NHANH:
Cho người mới (KHUYẾN NGHỊ):
✅ Dùng App Hik-Connect - Dễ nhất, không cần port forwarding
✅ Chỉ cần: Cắm camera → Tải app → Quét QR code → Xong!
Cho dự án AI (code Python):
✅ Setup DDNS + Port Forwarding
✅ Kết nối RTSP từ xa

⚠️ Một số lỗi thường gặp:
1. Không tìm thấy camera trong SADP:

Kiểm tra dây mạng đã cắm chặt chưa
Tắt Firewall Windows tạm thời
Đảm bảo laptop và camera cùng mạng

2. App Hik-Connect báo "Offline":

Kiểm tra camera có kết nối Internet không (test ping)
Kiểm tra đã bật "Platform Access" chưa
Thử xoá và thêm lại camera

3. Không truy cập được từ xa:

Kiểm tra router có IP Public (không phải IP nội bộ 10.x, 172.x, 192.168.x)
Kiểm tra ISP có chặn port không (gọi nhà mạng)
Thử đổi port khác (8081, 8082...)


📞 Cần trợ giúp?
Nếu bạn gặp khó khăn ở bước nào, hãy cho mình biết:

Router bạn dùng hãng gì? (TP-Link, Tenda, Modem nhà mạng?)
Báo lỗi cụ thể là gì?
Screenshot màn hình để mình hỗ trợ chi tiết hơn!