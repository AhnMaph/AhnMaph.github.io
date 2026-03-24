---
title: "Cach lock bootloader cho may Xiaomi / Redmi / Poco"
date: 2026-03-24 10:30:00 +0700
categories: [mobile, android]
tags: [xiaomi, redmi, poco, bootloader, fastboot, miflash]
pin: false
toc: true
---

> Luu y: Bài viết mang tính tham khảo. Mọi thao tác flash ROM/khoa bootloader đều có rủi ro, bạn cần tự chịu trách nhiệm và đối chiếu thêm tài liệu chính thống cho đúng model máy.

# WARNING

Việc chạy phần mềm qua Mi Flash ở chế độ `clean all and lock` sẽ **CÀY PHẲNG MỌI DỮ LIỆU** và thiết lập lại hệ thống phân vùng. Bạn BẮT BỘC phải rà soát 3 chốt chặn an toàn sau:

1. **Bảo vệ mã Authenticator / MFA (Rủi ro vĩnh viễn):** Khôi phục cài đặt gốc sẽ xóa sạch các app tạo mã (Google/Microsoft Authenticator, Authy...). Nếu bạn đang dùng chúng để bảo mật các tài khoản trọng yếu (AWS, Server, Binance, Google, Facebook...), **BẮT BUỘC phải Export (Chuyển mã) sang thiết bị khác, hoặc lưu trữ Backup Codes ra nơi an toàn**. Mất mã quét là mất vĩnh viễn quyền truy cập tài khoản.
    
2. **Khóa chống trộm (Activation Lock & FRP):** Đang dùng bình thường thì phải vào Cài đặt để **Đăng xuất sạch sẽ Tài khoản Mi** và **Xóa toàn bộ Tài khoản Google (Gmail)**. Kế tiếp, tắt Mật khẩu/Mã PIN màn hình. Nếu quên làm bước này, sau khi chạy ROM xong máy sẽ khóa cứng, đòi mật khẩu cũ.
    
3. **An toàn phần cứng & Nguồn điện:** Rủi ro "đột tử" (Hard Brick) xảy ra cao nhất nếu quá trình nạp Bootloader bị ngắt điện. Điện thoại phải còn **trên 50% pin**. Máy tính (Laptop) phải cắm sạc trực tiếp và chỉnh cài đặt màn hình sang chế độ **Never Sleep** (Không bao giờ ngủ).
### 🔍 PHẦN 2: TRUY TÌM TÊN MÃ MÁY (PRODUCT CODENAME) CHUẨN XÁC
Thế giới Xiaomi có hàng trăm mẫu máy với tên gọi na ná nhau (Ví dụ: Redmi Note 13 có cả bản 4G, 5G, Pro, Pro+...). Tên thương mại có thể lừa bạn, vỏ hộp có thể in sai, phần mềm bên trong có thể bị cửa hàng "độ chế", nhưng **Mã phần cứng (Product Code) thì không bao giờ nói dối**.

**Bước A: Kiểm tra sơ bộ trên điện thoại (Tham khảo)**
	**Bước 1: Tra cứu IMEI (Xác định thị trường xuất xứ)**
1. Mở ứng dụng Gọi điện, bấm cú pháp **`*#06#`** để lấy số IMEI 1 của máy.
    
2. Truy cập trang web kiểm tra sản phẩm chính hãng của Xiaomi (hoặc các trang như _imei24.com_). Nhập số IMEI vào để tra cứu.
    
3. **Cách đọc kết quả:**
    
    - Nếu web báo **"Official International Version"**: Chúc mừng, máy bạn sinh ra là Bản Quốc tế. Lựa chọn tuyệt vời và an toàn nhất là tải bản ROM Global.
        
    - Nếu web báo **chữ Trung Quốc** hoặc xuất xứ China: Máy bạn là Bản Nội địa. Lúc này bạn phải cực kỳ cẩn thận, TUYỆT ĐỐI KHÔNG dùng lệnh Khóa Bootloader (`clean all and lock`) nếu định cài ROM Global, nếu không máy sẽ bị Brick vĩnh viễn.
        
    - _⚠️ Cảnh báo:_ Tên hiển thị trên web check IMEI thường viết tắt (VD: chỉ ghi "Redmi Note 13", bạn ngầm hiểu là bản tiêu chuẩn 4G). Tên này KHÔNG được dùng để tìm file ROM tải về.

4. Vào **Cài đặt** > **Giới thiệu điện thoại** (About phone).
    
5. Xem dòng **Phiên bản MIUI/HyperOS**: Chú ý 7 chữ cái in hoa (Ví dụ: `OS1.0.2.0.UNHMIXM`). Chữ `MI` ở giữa cho biết đây là bản Global.
    
6. Vào **Tất cả thông số**: Ghi nhớ dòng **Tên thiết bị** (Device name) và **Số kiểu máy** (Model number).
**Bước B: Khám nghiệm phần cứng bằng Fastboot (Quyết định 100%)**
#### TẢI MI FLASH TOOL
Tải Miflash bản **2017.04.25** https://xiaomiflashtool.com/download/xiaomi-flash-tool-20170425 
Tải bản này sẽ đỡ lỗi hơn ví dụ timeout như sau
!(Ảnh lỗi)[images/error-blog2.png]
1. Giải nén file nén ROM đuôi `.tgz` (bản Global) ra thẳng ổ **C:**. Đổi tên thư mục thật ngắn gọn (Ví dụ: `C:\romchuan`). Mở thử ra xem có thư mục `images` và các file `.bat` bên trong là đúng
    
2. Mở phần mềm **Mi Flash Tool** (`XiaoMiFlash.exe`).
    
3. Tắt nguồn điện thoại. Bấm giữ **Nguồn + Giảm âm lượng** cho đến khi hiện chữ **FASTBOOT** màu cam. Cắm cáp vào máy tính.
    
4. Trên máy tính, mở thư mục chứa công cụ ADB/Fastboot (Ví dụ: `F:\MiFlash\Source\ThirdParty\Google\Android`).
    
5. Click lên thanh địa chỉ thư mục, gõ `cmd` rồi Enter để mở bảng đen.
    
6. Gõ lệnh soi mã gốc: **`fastboot getvar product`**
    
    - 👉 Kết quả trả về sau chữ `product:` chính là "chân ái" của bạn (Ví dụ: `sapphire`, `garnet`, `marble`...). Hãy ghi chép lại cái tên này cẩn thận.
        
7. Kiểm tra an toàn phụ trợ:
    
    - Gõ **`fastboot getvar anti`**: Nếu kết quả là khoảng trống, lỗi `FAILED`, hoặc số `1` -> Máy an toàn, không bị khóa hạ cấp.
        
    - Gõ **`fastboot getvar unlocked`**: Kết quả `yes` -> Máy đã mở khóa Bootloader, sẵn sàng nhận ROM.
        

---

### 🛒 PHẦN 3: TẢI ĐÚNG ROM CHO MÃ PRODUCT

1. Truy cập trang web uy tín (Ví dụ: `mifirm.net` hoặc `xiaomifirmwareupdater.com`).
    
2. Nhập chính xác cái tên mã bạn vừa lấy được từ bước B (Ví dụ: `sapphire`) vào ô tìm kiếm. Tuyệt đối không tự ý thêm bớt ký tự (không thêm chữ `n` hay gõ tên thương mại).
    
3. Sử dụng bộ lọc (Filter):
    
    - **Region (Khu vực):** Chọn đúng khu vực (ở bước 2) **Global, China, Taiwan v.v**         
    - **Type (Loại ROM):** Bắt buộc chọn **Fastboot**.
        
4. Tải file trên cùng (Mới nhất). Kiểm tra lại lần cuối: File tải về bắt buộc phải có đuôi là **`.tgz`** (nếu là `.zip` là sai loại, dành cho Recovery).
    
	[All Information About MIUI ROM Variants & Regions - Xiaomiui.Net](https://xiaomiui.net/all-information-about-miui-rom-variants-regions-4611/#google_vignette)
	[History](https://www.reddit.com/r/Xiaomi/comments/ojtgk6/how_to_fix_the_flash_timeout_error/)

---

### ⚙️ PHẦN 4: THIẾT LẬP MI FLASH TOOL (KHẮC PHỤC LỖI 700S)

1. **Chuẩn bị file:** Giải nén cục ROM `.tgz` ra ngay thư mục gốc của ổ **C:** hoặc **D:**. Đặt tên thư mục chứa ROM cực ngắn, viết liền không dấu (VD: `C:\romchuan`). Mở thư mục đó ra phải thấy ngay thư mục `images` và các file `.bat`.
    
2. **Mở Mi Flash Tool** (`XiaoMiFlash.exe`).
    
3. **Sửa lỗi ngắt kết nối (Timeout):**
    
    - Nhìn lên thanh menu trên cùng, click **Configuration** > **MiFlash Configuration**.
        
    - Tìm dòng **`Flash timeout`** (hoặc Check point). Xóa số `700` mặc định, nhập vào số **`3000`** > Bấm **OK**. (Bước này đảm bảo phần mềm không tự ngắt khi đang chép file ROM nặng).
        

---

### ⚡ PHẦN 5: CHỐT HẠ & KHÓA BOOTLOADER (ĐIỂM KHÔNG THỂ QUAY ĐẦU)
!(Giao diện)[images/normal-blog2.png]
1. Trên Mi Flash, bấm nút **Select**. Trỏ đường dẫn đến thư mục `C:\romchuan` (Tuyệt đối chỉ chọn thư mục cha chứa thư mục `images`, không chọn chui vào bên trong `images`).
    
2. 🔴 **THAO TÁC SỐNG CÒN:** Nhìn xuống góc dưới cùng bên phải phần mềm, dùng chuột click chọn đúng vào ô tròn **`clean all and lock`** (Xóa sạch và khóa Bootloader lại).
    
3. Bấm **Refresh**. Đảm bảo bảng trắng hiện ra dãy ký tự ở cột _device_ (nhận diện thành công).
    
4. Đảm bảo dây cáp cắm chặt, điện thoại nằm im. Bấm nút **Flash** ở góc trên cùng bên phải.
    
5. Khoanh tay lại và chờ đợi. Thanh màu xanh lá sẽ chạy liên tục.
    
![Success](images/success-blog2.png)

---

### 🔄 PHẦN 6: CHỜ ĐỢI FIRST BOOT VÀ NGHIỆM THU

1. Khi thanh màu xanh chạy hoàn tất, trạng thái báo **`success`**, điện thoại sẽ nhận lệnh sập nguồn và tự khởi động lại. Lúc này, bạn có thể an tâm rút cáp USB.
    
2. **GIAI ĐOẠN NHẠY CẢM:** Logo màn hình khởi động (Redmi/Xiaomi/HyperOS) sẽ hiện lên và đứng im rất lâu. **Tuyệt đối không can thiệp, không bấm giữ phím Nguồn.** Quá trình bung file hệ điều hành lần đầu tiên (First Boot) luôn mất từ **5 đến 15 phút**.
    
3. Hãy cắm sạc điện thoại trực tiếp vào ổ cắm tường để đảm bảo máy không sập nguồn giữa chừng lúc đang bung file.
    
4. Khi màn hình "Hello / Xin chào" hiện lên, bạn đã thành công 90%.
    
5. Mở chế độ nhà phát triển (vào mục Giới thiệu điện thoại) tìm mục Trạng thái mở khóa Mi
![Hinh 1](images/pic1-blog2.png)
![Hinh 2](images/pic1-blog2.png)

---
