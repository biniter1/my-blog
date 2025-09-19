---
date : '2025-09-18T22:41:39+07:00'
title : 'Hành trình săn bug: Case Study: Phát hiện dấu hiệu Subdomain Takeover — gsslink.agilebits.com → d41c4ucz81chh.cloudfront.net'
keyword : 'Bug hunting'
tags : ['Bug Hunting' , 'Subdomain Takeover' , 'CloudFront' , 'AWS' , 'DNS Security' , 'Web Security']
cover :
    image: "https://i.ibb.co/TMS9Mdmn/Gemini-Generated-Image-kw8jmckw8jmckw8j.png"
    alt: "Bug s1"
    hiddenInList: false 
---
---
## 📋 Tóm tắt

Trong lần thu thập thông tin (recon), tôi phát hiện `gsslink.agilebits.com` là một CNAME trỏ tới `d41c4ucz81chh.cloudfront.net`. Khi truy vấn HTTPS, server trả HTTP/2 404 với header `x-cache: Error from cloudfront` và body là trang lỗi nginx. 

Các bằng chứng này chỉ ra rằng CloudFront distribution hoặc origin không phục vụ nội dung hợp lệ cho hostname này — đây là dấu hiệu **potential subdomain takeover** (tiềm ẩn).

> ⚠️ **Lưu ý đạo đức/pháp lý**: Tất cả thao tác chỉ là read-only (dig, curl, openssl). Không có hành vi đăng ký/claim resource, không thử thay đổi/cấu hình dịch vụ bên thứ ba. Không cung cấp hướng dẫn khai thác.

---

## 🎯 Bối cảnh

**Subdomain phát hiện**: `gsslink.agilebits.com`

**DNS Resolution**: CNAME → CloudFront: `d41c4ucz81chh.cloudfront.net`

Việc trỏ CNAME tới `*.cloudfront.net` gợi ý subdomain từng (hoặc đang) sử dụng AWS CloudFront làm CDN/distribution. Nếu distribution hoặc origin không cấu hình để trả nội dung cho hostname đó (ví dụ origin bị xóa, distribution không chứa Alternate Domain Name phù hợp), subdomain có thể trở nên "orphaned" và xuất hiện rủi ro takeover.

---

## 🔍 Các lệnh kiểm tra (Read-only Evidence)

### 1. Kiểm tra DNS CNAME

```bash
$ dig +short gsslink.agilebits.com CNAME
d41c4ucz81chh.cloudfront.net.
```

### 2. Thử truy vấn DNS toàn bộ

```bash
$ dig gsslink.agilebits.com ANY +noall +answer
;; Connection to 172.26.96.1#53(172.26.96.1) for gsslink.agilebits.com failed: timed out.
;; no servers could be reached
;; Connection to 172.26.96.1#53(172.26.96.1) for gsslink.agilebits.com failed: timed out.
;; no servers could be reached
;; Connection to 172.26.96.1#53(172.26.96.1) for gsslink.agilebits.com failed: timed out.
;; no servers could be reached
```

> 📝 **Ghi chú**: Lỗi này do resolver cục bộ (172.26.96.1) không phản hồi — không ảnh hưởng kết luận vì CNAME đã thu được qua `dig +short`.

### 3. Kiểm tra HTTP Headers

```bash
$ curl -I -L https://gsslink.agilebits.com
HTTP/2 404
content-type: text/html
content-length: 146
server: nginx
date: Fri, 19 Sep 2025 02:15:27 GMT
x-cache: Error from cloudfront
via: 1.1 0ad8fd57a8360042531af664f971c84a.cloudfront.net (CloudFront)
x-amz-cf-pop: HAN50-P2
x-amz-cf-id: KKiMMabNsT6D83STkcLgxOUTFFGoItJSx_BZfO8lrS2Z-KcV0H5YmQ==
```

### 4. Kiểm tra Response Body

```bash
$ curl -sS https://gsslink.agilebits.com | head -n 60
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

### 5. Kiểm tra TLS Certificate

```bash
$ echo | openssl s_client -connect gsslink.agilebits.com:443 -servername gsslink.agilebits.com 2>/dev/null | openssl x509 -noout -text | sed -n '1,20p'
Certificate:
    ...
    Validity
        Not Before: Sep  9 00:00:00 2025 GMT
        Not After : Oct  8 23:59:59 2026 GMT
    Subject: CN=gslink.email.1password.com
    ...
```

> 🚨 **Quan trọng**: Certificate Subject CN là `gslink.email.1password.com` — không khớp với `gsslink.agilebits.com`.

---

## 🔎 Phân tích kết quả

### ✅ Bằng chứng cho thấy Potential Subdomain Takeover

| Bằng chứng | Giải thích |
|------------|------------|
| **CNAME → CloudFront** | `d41c4ucz81chh.cloudfront.net` xác nhận DNS đang trỏ subdomain tới CloudFront distribution |
| **HTTP/2 404 + x-cache Error** | CloudFront edge nhận request nhưng gặp lỗi khi truy origin hoặc distribution không phục vụ hostname |
| **Nginx Error Page** | Origin trả trang lỗi nginx thay vì nội dung hợp lệ |
| **TLS Certificate Mismatch** | Certificate cho `gslink.email.1password.com`, không phải `gsslink.agilebits.com` |

### 🎯 Kết luận phân tích

Đây là **evidence of potential subdomain takeover/orphaned CloudFront mapping**. Không đủ bằng chứng rằng takeover thành công đã xảy ra, nhưng đủ để khuyến nghị chủ sở hữu kiểm tra và khắc phục ngay lập tức.

---

## ⚠️ Đánh giá rủi ro và tác động

### Mức độ rủi ro
**Trung bình → Cao** (tùy thuộc subdomain từng phục vụ: assets, JS, API, form, redirects...)

### Khả năng tấn công
Nếu attacker có thể tạo origin/distribution phù hợp, họ có thể:
- Host nội dung độc hại dưới subdomain
- Thực hiện phishing attacks
- Hijack tài nguyên quan trọng (JS libraries)
- Redirect users tới trang độc hại

### Tác động thực tế
- **Nếu subdomain từng cung cấp tài nguyên quan trọng**: Hậu quả nghiêm trọng
- **Nếu chỉ là subdomain phụ/unused**: Ảnh hưởng ít hơn nhưng vẫn không nên để tồn tại

---

## 🛠️ Khuyến nghị khắc phục

### Nếu subdomain không còn sử dụng
- ❌ **Xóa record CNAME** khỏi DNS zone

### Nếu subdomain muốn tiếp tục sử dụng

#### 🔧 Cấu hình CloudFront
- ✅ Đảm bảo CloudFront distribution có **Alternate Domain Name (CNAME)** bao gồm `gsslink.agilebits.com`
- ✅ Gắn **chứng chỉ TLS phù hợp** (ACM cert) bao gồm hostname
- ✅ Kiểm tra **origin** (S3 bucket/origin server) tồn tại và trả nội dung cho hostname

#### 🗃️ Nếu origin là S3
- ✅ Đảm bảo bucket tồn tại
- ✅ Tên bucket/virtual-hosted-style đúng
- ✅ Policy/public access hợp lý (không public sensitive data)

#### 🔒 Bảo mật bổ sung
- ✅ Thêm security headers: HSTS, CSP, X-Content-Type-Options, X-Frame-Options
- ✅ **Giám sát DNS**: Tự động scan zone để phát hiện CNAME trỏ tới third-party domains
- ✅ **Audit & Checklist**: Bổ sung kiểm tra vào quy trình deploy/CI

---

## 📢 Disclosure Note

> 🔍 **Minh bạch với độc giả**: Mọi thao tác trong bài viết này chỉ là **passive/read-only**: dig, curl, openssl. Tôi không đăng ký, claim hoặc thay đổi bất kỳ tài nguyên nào của bên thứ ba. 
>
> Lỗ hỏng này có thể đã được xử lý (out-of-scope với chương trình bounty có liên quan). Bài viết nhằm mục đích **giáo dục** và chia sẻ quy trình kiểm chứng an toàn.

---

## 🎯 Kết luận

Các bằng chứng thu thập từ việc phân tích `gsslink.agilebits.com` tạo thành một bức tranh thuyết phục:

- **CNAME → CloudFront** với phản hồi lỗi
- **HTTP/2 404** với `x-cache: Error from cloudfront`
- **Nginx error page** thay vì nội dung hợp lệ  
- **Certificate mismatch** cho hostname khác

Tất cả đều chỉ ra rằng subdomain hiện **không được phục vụ nội dung hợp lệ** qua CloudFront — đây là **potential subdomain takeover**. 

**Chủ sở hữu nên kiểm tra và khắc phục ngay lập tức** theo các khuyến nghị đã nêu để đảm bảo an toàn hệ thống.

---


**Date**: September 18, 2025