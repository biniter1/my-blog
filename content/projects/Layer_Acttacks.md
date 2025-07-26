---
date : '2025-07-26T20:19:31+07:00'
title : 'Layer2_Acttacks'
tags : ['Layer_Acttacks','Attack','Projects','Hacking','CEH']
---

# Tấn Công Layer 2: Hiểu và Phòng Chống Các Cuộc Tấn Công MAC
Chào tất cả mọi người cảm ơn mọi người đã dành thời gian đọc blog này!
Trong bài viết này, chúng ta sẽ khám phá các cuộc tấn công Layer 2, cách chúng hoạt động và phòng chống
## Phần 1: Các khái niệm  
Trước khi đi vào tìm hiểu thì chúng ta sẽ làm rõ Layer 2-Tầng Data link là gì?  
Mỗi thiết bị vật lý đều có một địa chỉ được gắn vào NIC (Network Interface Card) của nó, địa chỉ này được gọi là MAC (Media Access Control) địa chỉ. 
![MAC](https://www.ipxo.com/app/uploads/2022/05/MAC-address-min.png)
Thì địa chỉ MAC gồm hai phân đoạn:
- 24 bit (6 byte) đầu cho **Organizationally Unique Identifier** (OUI) giúp phân biệt các nhà sản xuất.
- 24 bit (6 byte) còn lại cho thiết bị.  
  **Lưu ý**: địa chỉ MAC không thể thay đổi và gắn liền với thiết bị.

### Giao thức ARP là gì?
Là giao thức từ địa chỉ IP biết được địa chỉ MAC của máy tính khác.
![ARP](https://www.fortinet.com/content/dam/fortinet/images/cyberglossary/what-is-arp.jpg)

### Bảng CAM là gì?
Là bảng lưu trữ địa chỉ MAC của các thiết bị trong mạng.
![CAM](https://www.computernetworkingnotes.com/wp-content/uploads/ccna-study-guide/images/csg177-01-cam-table.png)
## Phần 2: Các loại tấn công MAC
### 1. Tấn công MAC Flooding
Như đã nói ở trên bảng CAM sẽ lưu lại địa chỉ mỗi khi một thiết bị kết nối vào mạng. Nhưng chuyện gì sẽ xảy ra nếu một thiết bị gửi một lượng lớn địa chỉ MAC giả vào bảng CAM?  
![MAC Flooding](https://app.trustline.sa/media/blog/2024/10/09/scrnn4.jpeg)
- **Lỗi**: Bảng CAM sẽ đầy và không thể lưu thêm địa chỉ MAC nào nữa.
- **Hiệu ứng**: Mạng sẽ bị quá tải và không thể kết nối được với các thiết bị hợp lệ. Switch bị quá tải không thể xử lý được yêu cầu từ đó các gói tin sẽ được switch gửi broadcast (được gửi đến toàn bộ thiết bị trong mạng ) và máy attacker sẽ bắt được toàn bộ những gói tin này từ đó có thể đọc được thông tin của các thiết bị trong mạng rồi thực hiện tiếp các cuộc tấn công khác
### 2. Tấn công MAC Spoofing
Đây là một cuộc tấn công attacker sẽ chen chân vào giữa đường truyền.
![MAC Spoofing](https://www.securew2.com/wp-content/uploads/2023/05/mac_spoofing-1024x310.png)
Chúng sẽ gửi một gói tin đến máy nạn nhận bảo tôi là **gateway của bạn** xong lại gửi 1 gói tin cho gateway bảo **tôi là máy A(victim)** vậy từ giờ khi máy A gửi gói tin đi đều đi trung gian qua máy attacker.  
Vậy chúng có thể sửa thay đổi làm bất kỳ chúng muốn.   
**Một dạng tấn công khác**: chúng giả dạng làm máy nạn nhận khiến switch không biết máy nào là thật gây xung đột.

## Phần 3: Phòng chống 
### 1. Chống MAC Flooding với Port Security
Chúng ta sẽ sử dụng tính năng Port Security trên switch chỉ cho phép học MAC của một thiết bị duy nhất trên mỗi cổng hoặc giới hạn số thiết bị học trên 1 cổng. 
```Cisco CLI
! Đi vào chế độ cấu hình interface
interface FastEthernet0/1

! Đảm bảo cổng đang ở mode access
switchport mode access

! Bật tính năng Port Security trên cổng này
switchport port-security

! Giới hạn chỉ cho phép tối đa 2 địa chỉ MAC học trên cổng
! (1 cho máy tính, 1 có thể dự phòng cho điện thoại IP chẳng hạn)
switchport port-security maximum 2

! Nếu có vi phạm, cổng sẽ tự động bị "shutdown"
switchport port-security violation shutdown

! (Tùy chọn) Tự động học MAC đầu tiên và "gắn" nó vào cấu hình
! Giúp chống việc ai đó rút máy tính ra và cắm thiết bị khác vào
switchport port-security mac-address sticky

! Thoát khỏi chế độ cấu hình
exit
```
### 2. Chống Giả mạo và Đầu độc ARP với DAI & DHCP Snooping
- **DAI (Dynamic ARP Inspection)**: Chặn các gói tin ARP giả mạo.
```Cisco CLI
! Bật DAI trên các VLAN tương ứng
ip arp inspection vlan 10,20

! --- Cấu hình cổng tin cậy (Trusted Port) ---
! Tương tự DHCP Snooping, bạn cần khai báo cổng nối đến router là tin cậy.
! DAI sẽ không kiểm tra các gói tin ARP đến từ cổng này.
interface GigabitEthernet0/1
    ip arp inspection trust
exit

! --- Các cổng còn lại (Untrusted Port) ---
! DAI sẽ kiểm tra TẤT CẢ các gói tin ARP đến từ các cổng không tin cậy.
! Nếu thông tin trong gói tin ARP không khớp với CSDL của DHCP Snooping,
! gói tin đó sẽ bị hủy bỏ.
```
- **DHCP Snooping**: Chặn các gói tin DHCP giả mạo.
```Cisco CLI
! Bật DHCP Snooping trên toàn bộ switch
ip dhcp snooping

! Chỉ định VLAN mà bạn muốn DHCP Snooping hoạt động
ip dhcp snooping vlan 10,20

! --- Cấu hình cổng tin cậy (Trusted Port) ---
! Đây là cổng nối đến máy chủ DHCP hoặc router cấp phát IP
interface GigabitEthernet0/1
    ip dhcp snooping trust
exit

! --- Các cổng còn lại (Untrusted Port) mặc định không cần cấu hình gì thêm ---
! Switch sẽ tự động coi các cổng không được khai báo "trust" là "untrus
```
---
Nếu các bạn muốn demo thực hiện các cuộc tấn công trên thì các bạn hãy sử dụng phần GNS3 để mô phỏng quá trình trên để hiểu rõ chi tiết hơn.




