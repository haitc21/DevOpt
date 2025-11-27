# Setup Domain

## 1. Mua Domain

- Để mua được tên miền thì cần có thẻ thanh toán quốc tế VISA hoặc MasterCard.
- Truy cập [GoDaddy](https://www.godaddy.com/en/offers/godaddy)
- Tìm domain mình muốn. Ví dụ ở đây làm lab sẽ dùng **devopshaitc.online** với giá 35k cả VAT.

## 2. Kết nối domain đến CloudFlare

- Trên [GoDaddy](https://www.godaddy.com/en/offers/godaddy) nhấn vào User Name trên gốc trên bên phải => My Products => Domain => DNS => Nameservers => Change Nameserver
- Đăng nhập vào [CloudFlare](https://www.cloudflare.com)
- Nhập domain vừa mua ở trên vào ô tìm kiếm của CloudFlare.
- Copy 2 Nameserver từ CloudFlare thay cho Nameserver mặc định của GoDaddy.
- Việc cấu hình Nameserver này thì CloudFlare cần 1 khoảng thời gian để cập nhật nên cần đợi (thông thường khoảng 14p).
