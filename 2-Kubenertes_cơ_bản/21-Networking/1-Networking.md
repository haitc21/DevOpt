# Kết nối giữa các Pod trong Kubernetes và Sử dụng NetworkPolicy

Trong Kubernetes, các **Pod** không nên gọi trực tiếp nhau qua địa chỉ IP, vì mỗi lần Pod restart, IP có thể thay đổi.  
Thay vào đó, ta sử dụng **Service** để cung cấp một điểm truy cập ổn định.  
Khi cần giới hạn kết nối giữa các Pod, ta dùng **NetworkPolicy**.

---

## 1. Cú pháp DNS của Service

Kubernetes tự động cung cấp DNS nội bộ cho mỗi Service.

Cú pháp đầy đủ:

```
<ServiceName>.<Namespace>.svc.cluster.local
```

Nguồn:  
[Kubernetes DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

Ví dụ:  
Service `redis-sentinel` trong namespace `architecture` sẽ có tên DNS:

```
redis-sentinel.architecture.svc.cluster.local 
```

Nếu các Pod nằm **trong cùng namespace**, bạn chỉ cần gọi ngắn gọn:

```
redis-sentinel
```

## 2. NetworkPolicy

**NetworkPolicy** là "tường lửa" (firewall) ở mức Pod, giúp bạn kiểm soát:

- Pod nào được phép **gửi (Egress)** traffic ra ngoài
- Pod nào được phép **nhận (Ingress)** traffic vào trong

> Lưu ý: NetworkPolicy chỉ hoạt động nếu **Network Plugin (CNI)** hỗ trợ, ví dụ: Calico, Cilium, Weave Net.

---

### Hai hướng traffic

| Loại | Ý nghĩa |
|------|----------|
| **Ingress** | Cho phép traffic *đi vào* Pod |
| **Egress** | Cho phép traffic *đi ra* từ Pod |

Mặc định, Pod **được phép tất cả** (Ingress + Egress).  
Khi tạo NetworkPolicy, bạn sẽ quy định cụ thể traffic nào được phép.

---

### Ví dụ: Cho phép backend gọi redis

Bạn muốn:

- `ecommerce-frontend` → gọi `ecommerce-backend`
- `ecommerce-backend` → gọi `redis-sentinel`
- `ecommerce-frontend` ❌ không được gọi `redis-sentinel`

### NetworkPolicy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-policy
  namespace: architecture   # Nơi có pod redis-sentinel
spec:
  podSelector:
    matchLabels:
      app: redis-sentinel   # Chọn pod redis cần bảo vệ
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ecommerce   # namespace chứa backend
      podSelector:
        matchLabels:
          app: ecommerce-backend  # chỉ backend mới được phép gọi
    ports:
    - protocol: TCP
      port: 6379

```

Giải thích:

- Chọn Pod có label `app=ecommerce`
- Chỉ cho phép kết nối **từ Pod có label `app=ecommerce-backend`**
- Chỉ cho phép cổng 6379 (redis)

Kết quả:
✅ Backend gọi Redis được  
❌ Frontend không gọi Redis được

---

### Default Deny (mặc định chặn hết)

Nếu bạn muốn chặn tất cả traffic và chỉ mở dần từng phần, hãy dùng “default deny”.

- Chặn toàn bộ Ingress

```yaml
podSelector: {}
policyTypes:
- Ingress
```

- Chặn toàn bộ Egress

```yaml
podSelector: {}
policyTypes:
- Egress
```

- Chặn cả Ingress và Egress

```yaml
podSelector: {}
policyTypes:
- Ingress
- Egress
```

---

## 3. Tổng kết nhanh cho developer

| Mục đích | Cách làm |
|-----------|----------|
| Pod nói chuyện với Pod khác | Dùng `Service` |
| Truy cập ổn định qua DNS | `<Service>.<Namespace>.svc.cluster.local` |
| Kiểm soát traffic | Dùng `NetworkPolicy` |
| Giới hạn chiều Ingress/Egress | Quy định `policyTypes` |
| Chặn tất cả trước rồi mở dần | “Default deny” policy |

---
