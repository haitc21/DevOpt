# Phân quyền trong Kibana

## 1. Tạo Space

![](./images/1.png)

Tạo Space `dev-team`

![](./images/2.png)

Phần `Solution View` là giới hạn chức năng hiển thị trên giao diện Menu của Space.
Chọn `Classic` để có thể tùy chỉnh `Set feature visibility`

![](./images/3.png)

Tương tự chúng ta có thể tạo space cho các team khác như BA, Tester, Networking, SA v.v.

## 2. Tạo Role và User

Cuyển về Space `default` để có các Menu cấu hình.
Vào `Stack Management` => `Roles`, Kibana đã tạo sẵn một số Role mặc định

![](./images/4.png)

Để rõ ràng hơn, chúng ta cần 1 Role để gán cho dev team của dự án ecommerce.

![](./images/5.png)

Gán Role vào Space `dev-team` và thiết lập quyền cho từng chức năng

![](./images/6.png)

Tạo user `ecommerce-dev`

![](./images/7.png)