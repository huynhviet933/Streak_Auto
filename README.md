# HƯỚNG DẪN CHI TIẾT SỬ DỤNG STREAK.APP AUTO TOOL

Công cụ này giúp tự động hóa quy trình: Đăng nhập ví -> Xác thực Email (KYC) -> Đổi tên người dùng (Username) -> Điểm danh nhận XP (Check-in).

---

### 1. YÊU CẦU CÀI ĐẶT
Máy tính cần cài đặt Node.js. Sau đó, mở Terminal tại thư mục tool và chạy lệnh:
`npm install fs-extra axios ethers@5.7.2 uuid colors https-proxy-agent luxon node-machine-id`

---

### 2. CẤU TRÚC CÁC FILE ĐẦU VÀO (INPUT)
Bạn cần chuẩn bị các file sau trong thư mục gốc của tool:

1. **privatekey.txt**: Danh sách Private Key ví (mỗi dòng 1 key).
   - Ví dụ: `0xabc123...`
2. **proxy.txt**: Danh sách Proxy để đổi IP (mỗi dòng 1 proxy).
   - Định dạng: `http://user:pass@ip:port` hoặc `ip:port`.
3. **user_agents.txt**: Danh sách trình duyệt giả lập.
   - Nếu không có, tool sẽ tự dùng mặc định.
4. **mail.txt**: Danh sách tài khoản Mail.tm có sẵn.
   - Định dạng bắt buộc: `email|password|token`
   - Tool sẽ lấy mail theo thứ tự tương ứng với số thứ tự ví.
5. **Config.json**: Cấu hình luồng và thời gian nghỉ.
   - `maxThreads`: Số ví chạy cùng lúc.
   - `staggerDelay`: Khoảng cách giữa các luồng (giây).
   - `accountDelay`: Thời gian nghỉ sau khi xong 1 ví (giây).

---

### 3. CÁC FILE ĐẦU RA VÀ QUẢN LÝ DỮ LIỆU (OUTPUT)
Sau khi tool chạy, các file sau sẽ xuất hiện:

1. **license.txt**: Lưu License Key bạn đã nhập lần đầu. Tool dùng HWID để khóa máy.
2. **profiles.json**: Đây là file quan trọng nhất. 
   - Lưu trữ toàn bộ thông tin: UID, Email, Username, Tổng XP đã cày được.
   - Lưu trữ User-Agent riêng cho từng ví để đảm bảo không bị đổi môi trường đăng nhập.
3. **index.txt**: Lưu lại vị trí ví cuối cùng đang chạy. 
   - Nếu tool bị ngắt, lần sau mở lại nó sẽ tự động chạy tiếp từ ví đó (Resume).
   - Nếu qua Ngày Mới Tool Không chạy thì xóa File Này Là sẽ Chạy

---

### 4. QUY TRÌNH HOẠT ĐỘNG CỦA TOOL
- **Bước 1 (License)**: Xác thực bản quyền qua Server.
- **Bước 2 (Login)**: Ký thông báo (Sign Message) bằng ví để lấy Session Cookie.
- **Bước 3 (KYC Mail)**: Nếu ví chưa có mail, tool lấy 1 dòng trong `mail.txt`, gửi mã OTP về và tự động giải mã qua Token Mail.tm.
- **Bước 4 (Rename)**: Nếu tên là mặc định (`User_...`), tool tự đổi sang tên người thật theo định dạng `ten_hoX` (Ví dụ: `yen_nhi5`, `minh_phan2`).
- **Bước 5 (Check-in)**: Gửi lệnh Claim XP hằng ngày.
- **Bước 6 (Save)**: Lưu toàn bộ trạng thái vào `profiles.json` và in log thành công.

---

### 5. CÁC TÍNH NĂNG ĐẶC BIỆT
- **Username Real**: Tên người dùng được tạo ngẫu nhiên từ bộ từ điển Họ/Tên Việt Nam và Quốc tế, giúp tài khoản nhìn tự nhiên như người thật.
- **Lite Logging**: Tool chỉ hiển thị các bước thành công (Login, Rename, XP, DONE). Các lỗi Proxy, Timeout hoặc Mail 429 sẽ được xử lý im lặng để tránh rác màn hình.
- **Retry Mechanism**: Tự động đợi 10 giây và thử lại nếu API Mail.tm bị giới hạn (429).
- **Manual Cookie**: Tự quản lý Cookie thủ công, không bị xung đột với Proxy Agent, giúp tool chạy cực kỳ ổn định trên số lượng ví lớn.

---

### 6. CÁCH CHẠY TOOL
Lệnh khởi chạy: `node p1.js`
- Lần đầu chạy: Nhập License Key khi được yêu cầu.
- Các lần sau: Tool sẽ tự động chạy theo cấu hình trong các file `.txt`.
