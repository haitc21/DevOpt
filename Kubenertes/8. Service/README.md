# k8s Service

- Là 1 đối tượng trong k8s dùng để định nghĩa cách tiếp cần đến các Pod
- Điều phối traffic đến các Pod
- Service Type:
  - **Cluster IP**: Tạo địa chỉ IP nội bộ trong cụm. Các service bên ngoài muốn kết nối đến phải đi qua **Ingress** hoặc **Gateway**.
  - **Node Port**: Mở trực tiếp 1 kết nối từ Pod ra bên ngoài không cần đi qua **Ingress** hay **Gateway**.
  - **LoadBalancer**: Dùng cho Cloud Provider để điều phối traffic đến Pod.
  - **ExternalName**: Điều phối đến domain.

## 1. Node Port

- Kết nối trực tiếp đến Pod.
- Chỉ sử dụng được các port trong khoảng **30000 - 32767**
