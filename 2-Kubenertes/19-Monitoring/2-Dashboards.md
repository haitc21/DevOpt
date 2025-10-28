# Dashboards giám sát hệ thống k8s

- Khi cài đặt P&G Stack thì đã có sẵn một số Dashboards giám sát

![](./images/9.png)

- Khi thấy hệ thống bị chậm có thể xem dashboard `Kubernetes / API server`

![](./images/10.png)

- Ví dụ dự án `ecommerce` bị chậm thì cần tìm xét cá yếu tố
  -Tài nguyên: CPU, Memory.
  - Mạng: bandwidth
  - Kết nối DB, query DB....
  - Cache
- Trong dashboard dựng sẵn sẽ xét đượ 2 yếu tố là tài nguyên và mạng.
- Với tài nguyên dùng dashboard  `Kubernetes / Compute Resources / Namespace (Workloads)`chọn namespace `ecommerce`

![](./images/11.png)

- Đổi múi giờ sang `Asian/Ho Chi Minh`


![](./images/12.png)

- Kiểm tra Pod đang cao tải `Kubernetes / Compute Resources / Pod`

![](./images/13.png)

- `Kubernetes / Compute Resources / Workload`  cũng có thể xem lượng tài nguyên của các pod.

![](./images/14.png)

- `Kubernetes / Compute Resources / Cluster` dashboard tổng quan overview tài nguyên toàn cluser.

![](./images/15.png)

- Phần Networking tượng tự.