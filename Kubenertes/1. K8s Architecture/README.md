# Kiến trúc k8s

![](https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg)

## 1. pod

- Là đơn vị nhỏ nhất trong k8s
- Có thể hiểu pood như 1 người nhân viên
- Trong pod có thể chứa nhiều container. Có thể hiểu container như các kỹ năng của người nhân viên đó. Thông thường thì **1 pod chỉ chứa 1 container**

## 2. kubelet

- Thành phần **nhận yêu cầu từ kube-api-server để thực thi các Pod trên Node**
- Giống như trưởng phòng sẽ giao và giám sát công việc của nhân viên trong phòng

## 3. kube-proxy

- Thành phần **network** chạy trên mỗi node cho phép các pod giao tiếp với nhau và giao tiếp ra bên ngoài.

## 5. node

- 1 cluster chứa nhiều node.
- Node chứa: 1 kubelet, 1 kube-proxy và các pod
- Có thể hiểu node là phòng ban

## 6. Control Plane

- Có thể hiểu là ban giám đốc.
- Thường thì **server triển khai control plane thì không triển khai node**.

## 7 cloud-control-manager

- Chỉ trên Cloud mới có
- Có thể hiểu đây là ông giám đốc.

## 8. kube-api-server

- Là 1 quy chuẩn để giao tiếp giữa các yêu cầu.
- Kiểu như định nghĩa ra các api để giao tiếp. Ví dụ: Api để lấy danh sách node trong cluster là GET /api/v1/nodes

## 10. etcd

- Là 1 **CSDL** phân tán lưu trữ mọi thông tin, trạng thái của các thành phần trong cụm.

## 11. scheduler

- Chịu trách nhiệm phân phối pod đến các node trong cluster.
- Xem xét các yếu tố như: tài nguyên (cpu, memory), chính sách (policies), các yêu cầu cụ thể khác...
- Ví dụ chỉ định pod 1 vào node 1
- Chạy thuật toán tối ưu hóa để phân bổ workload: xem xét nên đặt pod vào node nào. Tương tự như đưa 1 nhân sự có chuyên môn vào phòng ban phù hợp.

## 12. controller manager

- Là những tiến trình để giám sát trạng thái của cluster và thực hiện các sửa chữa nếu cần.
- scheduler đưa pod vòa node => controller manager quản lý node đó, nếu node bị lỗi thì controller manager sẽ tự tạo 1 pod mới thay thế.
