# k8s {Pods

## 1. Lý thuyết

### 1.1 Giới thiệu

- Pod là đơn vị nhỏ nhất có thể triển khai và quản lý trong Kubernetes
- Pod có thể chứa init containers (chạy khi Pod khởi động) và ephemeral containers (dùng để debug).
- Pod context bao gồm Linux namespaces, cgroups và các cơ chế isolation khác, tương tự như container.

- Có hai cách dùng chính của Pod:

  - 1. Pod chạy một container duy nhất (phổ biến nhất, Pod như wrapper cho container).
  - 2. Pod chạy nhiều container cần phối hợp, chia sẻ tài nguyên (advanced use case).

### 1.2 Workload resources - quản lý Pods

- Thường không tạo Pod trực tiếp, kể cả singleton Pod. Thay vào đó dùng workload resources như Deployment, Job, StatefulSet.
- Mỗi Pod chạy một instance của ứng dụng. Để Horizontal scale (chiều ngang) → tạo nhiều Pod (replica).
- Workload resources và controllers quản lý replication, rollout (rình triển khai phiên bản mới của ứng), auto-healing (tự động phục hồi khi có lỗi ).

### 1.3 Làm việc với Pods

- Pods là ephemeral, disposable entities (ngắn hạn, dùng rồi bỏ).
- Khi được tạo, Pod được scheduler gán vào một Node và tồn tại cho đến khi:
  - Pod finishes execution
  - Pod object bị xóa
  - Pod bị evicted do thiếu resource
  - Node fails
    → (nghĩa là Pod không được migrate sang node khác, thay vào đó controller sẽ tạo Pod mới).
- Restart container ≠ Restart Pod (Pod là environment, không phải process).

### 1.4 Pod OS

- Trường `.spec.os.name` = `windows` hoặc `linux` (2 OS duy nhất được hỗ trợ).
- Mặc định Pod chạy trên Linux.
- Nếu cluster có nhiều OS, cần gán nhãn `kubernetes.io/os=<os>` cho từng node, ví dụ:

  ```yaml
  nodeSelector:
    kubernetes.io/os: windows
  ```

- Scheduler chọn node dựa trên nhiều yếu tố, không chỉ OS. OS này cần tương thích với container image (Windows container chạy trên node Windows).

### 1.5 Pods và controllers

- Workload resources (Deployment, StatefulSet, DaemonSet…) tạo và quản lý Pods.
- Dùng PodTemplate để định nghĩa Pods. PodTemplate không phải là file riêng, mà là một phần của workload resource (ví dụ trong file YAML của Deployment).
- Thay đổi PodTemplate thường dẫn đến việc tạo Pods mới thay thế, do hầu hết các cấu hình của Pod là immutable.

### 1.6 Pod update và replacement

- Một số field có thể update trực tiếp (`spec.containers[*].image`, `spec.tolerations`…).
- Phần lớn field là immutable, nên khi cần thay đổi → controller tạo Pod mới dựa trên PodTemplate.
- Subresources có thể thay đổi thêm:
  - resize: thay đổi resources
  - ephemeralContainers: thêm container tạm
  - status: cập nhật trạng thái
  - binding: gán Pod vào node

### 1.7 Pod generation

- `metadata.generation` tăng mỗi khi spec thay đổi.
- `status.observedGeneration` phản ánh generation hiện tại mà kubelet giám sát.

### 1.8 Resource sharing và communication

**Storage**

- Pod có thể khai báo shared volumes.
- Containers trong Pod truy cập volumes để chia sẻ dữ liệu và giữ data khi container restart.

**Networking**

- Mỗi Pod có IP riêng.
- Containers trong Pod chia sẻ network namespace, giao tiếp qua `localhost`.
- Các container trong Pod chung hostname = Pod name.
- Giao tiếp giữa Pods qua IP networking.

### 1.9 Pod security settings

- Dùng `securityContext` để giới hạn bảo mật:
  - Drop Linux capabilities (tắt bớt quyền trong kernel để giảm rủi ro bảo mật).
  - Chạy non-root user hoặc với user/group ID cụ thể.
  - Thiết lập seccomp profile (lọc system calls).
  - Cấu hình security cho Windows (ví dụ HostProcess).
    > ⚠️ `privileged mode` (container có toàn quyền như root host) chỉ dùng khi bắt buộc.

### 1.10 Static Pods

- Static Pod được quản lý trực tiếp bởi kubelet trên một node, không thông qua API server.
- Thường dùng để chạy control plane tự quản (self-hosted control plane).
- Kubelet tạo mirror Pod để hiển thị trên API server nhưng không điều khiển được từ đó.
- Spec của static Pod không tham chiếu các API objects khác.

### 1.11 Pods với nhiều containers

- Pod hỗ trợ nhiều container phối hợp thành một dịch vụ.
- Hai dạng chính:
  - Pod một container (phổ biến nhất).
  - Pod nhiều container gắn kết (ví dụ: sidecar).
- **Sidecar**: container phụ, hỗ trợ container chính (ví dụ: cập nhật file, proxy, service mesh).
- Pod có thể có init containers và sidecar containers.
- Từ v1.33: `SidecarContainers` cho phép init containers có `restartPolicy: Always`, hoạt động như sidecar.

### 1.12 Container probes

- Probe = kiểm tra định kỳ do kubelet thực hiện trên container.
- Loại probe:
  - ExecAction: chạy lệnh trong container.
  - TCPSocketAction: kiểm tra TCP socket.
  - HTTPGetAction: gửi HTTP GET.

# 2. Ví dụ triển khai tực tiếp Pod (chỉ để test ít sử dụng)

- Tạo file pod.yml

````sh
mkdir -p ~/projects/car-serv && cd ~/projects/car-serv
```

- Tạo namespace

````sh
vi ns.yml
```

```yml
apiVersion: v1
kind: Namespace
metadata:
  name: car-serv
```

````sh
kubectl apply -f ns.yml
```

- pod.yml

````sh
vi pod.yml
```

```yml
apiVersion: v1
kind: Pod
metadata:
  name: car-serv
  namespace: car-serv
spec:
  containers:
    - name: car-serv
      image: elroydevops/car-serv
      ports:
        - containerPort: 80
```

````sh
kubectl apply -f pod.yml
```

- Kiểm tra

````sh
kubectl get po -n car-serv
```

- Truy cập vào pod: Trong pod là container nên exec cũng tương tự docker

````sh
kubectl exec -it car-serv -n car-serv -- /bin/bash
```
