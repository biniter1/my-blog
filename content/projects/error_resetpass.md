---
date : '2025-07-28T20:23:35+07:00'
title : 'Lỗi Logic Chết Người Trong Chức Năng Reset Password: Khi $_REQUEST Phản Bội'
description : 'Lỗi logic chết người trong chức năng reset password: Khi $_REQUEST phản bội'
tags : ['lỗi logic', 'reset password', '$_REQUEST','Error']
cover:
    image: "https://itsystems.vn/wp-content/uploads/2021/05/0.jpg.webp"
    alt: ""
    hiddenInList: false
---
Ai trong chúng ta cũng đã từng nhấn vào nút "Quên mật khẩu". Nó là một tính năng tiện lợi và quen thuộc, một cứu cánh khi trí nhớ phản bội. Nhưng đằng sau sự đơn giản đó có thể ẩn giấu những lỗ hổng logic tinh vi, có khả năng biến một tính năng hỗ trợ thành cửa ngõ cho kẻ tấn công chiếm đoạt tài khoản.

Hôm nay, chúng ta sẽ cùng "mổ xẻ" một kịch bản tấn công kinh điển, khai thác lỗi logic trong quy trình đặt lại mật khẩu, chỉ vì một lựa chọn lập trình tưởng chừng như vô hại trong PHP: biến $_REQUEST.

Lỗi Logic là gì? 🕵️‍♂️
![Lỗi Logic](https://tryhackme-images.s3.amazonaws.com/user-uploads/5efe36fb68daf465530ca761/room-content/58e63d7810ac4b23051e1dd4a24ef792.png)
Trước tiên, hãy làm rõ khái niệm. Khác với các lỗ hổng kỹ thuật như SQL Injection hay XSS, lỗi logic không tấn công vào cú pháp của mã lệnh. Thay vào đó, nó khai thác những sai sót trong luồng xử lý và quy trình hoạt động của ứng dụng. Kẻ tấn công không "phá" code, mà "lừa" ứng dụng tự làm những điều mà lẽ ra nó không được phép.

Một ví dụ đơn giản: Một trang web kiểm tra quyền admin bằng cách xem URL có bắt đầu bằng /admin không. Nếu kẻ tấn công truy cập vào /adMin (viết hoa chữ M), hệ thống có thể bỏ qua bước kiểm tra và cho phép truy cập, đó chính là một lỗi logic.

---
**"Mổ xẻ" Lỗ hổng trong chức năng Reset Password**  
Chúng ta sẽ chia quy trình reset password thành hai đoạn như sau:
- Người dùng email muốn đặt lại mật khẩu gửi yêu cầu đến hệ thống.
- Hệ thống gửi một liên kết đến email người dùng, khi người dùng nhấp vào liên kết đó sẽ dẫn đến một nơi để thay đổi mật khẩu.

**Vậy chúng sẽ lợi dụng điều gì trong đây?**   
- Email muốn đặt lại được gửi theo phương thức GET
- Tên đăng nhập... được gửi theo phương thức POST 
Và đây máy chủ sử dụng biến $_REQUEST để lấy dữ liệu từ người dùng. Vậy nếu tại đây chúng thêm 1 trường email vào trong form và gửi theo phương thức POST thì sao? Server sẽ lấy email nào để gửi liên kết đó đi? Đó chính là trường email được nhập theo phương thức POST ưu tiên hơn. 

**Quy trình tấn công sẽ như sau:**
- Kẻ tấn công nhập email người dùng **victim@gmail.com** vào và yêu cầu reset password. Server thấy hợp lệ và chấp nhập.
```CLI
 curl 'http://domain/customers/reset?email=victim%40acmeitsupport.thm' -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=victim'
```
- Sau đưa đến 1 form để nhập thông tin thì chúng thêm 1 trường vào **username=victim&email=attacker@gmail.com** sau đó server lấy dữ liệu từ biến $_REQUEST là email của attacker
```CLI
curl 'http://domain/customers/reset?email=victim%40acmeitsupport.thm' -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=victim&email=attacker@hacker.com'
```
- Sau đó link liên kết được gửi đến chúng và tài khoản chúng ta đã bị chúng chiếm đoạt

---

**Làm thế nào để phòng tránh? 🛡️**  
Lỗi lầm này hoàn toàn có thể tránh được bằng cách tuân thủ các nguyên tắc lập trình an toàn.

Không bao giờ tin tưởng $_REQUEST: Nguyên tắc vàng là hãy luôn sử dụng các biến toàn cục cụ thể. Việc này giúp mã nguồn của bạn rõ ràng và an toàn hơn.

Sử dụng đúng biến cho đúng mục đích:

- Dùng $_GET để lấy dữ liệu từ URL (query string).

- Dùng $_POST để lấy dữ liệu được gửi lên từ các form.
