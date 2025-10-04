# k8s Deployment

## 1. Giới thiệu

- Là 1 trong các thành phần quản lý **Worklod** trong k8s
- Là thành phần được sử dụng nhiều nhất trong triển khai ứng dụng trên k8s, hỗ trợ quản lý pod, replica, auto stacle, limit resource...
- Quản lý phiên bản ứng dụng: Khi nâng phiên bản image, Deployment cũng lưu lại lịch sử.
- Tự động khắc phục lỗi. Ví dụ cấu hình Deployment có 3 pod nếu 1 Pod bị lỗi thì k8s sẽ tự động tạo Pod mới

## 2. Tạo Deployment

- Có thể tự viết file yml như các bài trước tuy nhiên để nhanh và đúng chuẩn thì nên sử dụng công cụ UI để tạo trực tiếp hoặc gen ra file yml
- vào [Rancher](https://rancher.local)
- Chọn cụm **devopseduvn** tạo ở bài [cài đặt rancher](../5.%20Rancher/README.md)
- Chọn namespace () **car-serv** tạo ở bài ![Pod](../6.%20Pods/README.md)
- Bên thanh Sidebar chọn Workloads => Deployments => Create
- Cấu hình:
  - Namespace: car-serv
  - ame: car-serv-deployment
  - Replica: 3
  - Container name: car-serv
  - Image: docker.io/elroydevops/car-serv:latest
  - Add Pod: tcp, 80, TCP

![](./images/1.png)

- Nhấn Create và chờ. Kết quả:

![](./images/2.png)

- Tạo được 3 pod trên 2 worker là sv2 và sv3. Địa chỉ Ip này chỉ là Internal IP để các Pod kết nối nội bộ.
- Nhấn vào dấu ... ở dòng car-serv-deployment chọn **Download YAML**

![](./images/3.png)

- Mờ file [car-serv-deployment.yaml](./car-serv-deployment.yaml) chỉnh sửa:
  - Trong metadata xóa:
    - metadata:
      - creationTimestamp
      - generation
      - managedFields
      - resourceVersion
      - uid
  - Trong spec xóa:
    - progressDeadlineSeconds
    - strategy
    - template:
      - metadata:
        - creationTimestamp
    - spec:
      - containers:
        - resources
        - securityContext
        - securityContext
        - terminationMessagePath
        - terminationMessagePolicy
      - dnsPolicy
      - restartPolicy
      - schedulerName
      - securityContext
      - terminationGracePeriodSeconds
  - Xóa hết phần status
- Kết quả thu được file deployment.yml cơ bản nhất đã loại bỏ hết các giá trị mặc định và tự động sinh

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    workload.user.cattle.io/workloadselector: apps.deployment-car-serv-car-serv-deployment
  name: car-serv-deployment
  namespace: car-serv
spec:
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      workload.user.cattle.io/workloadselector: apps.deployment-car-serv-car-serv-deployment
  template:
    metadata:
      labels:
        workload.user.cattle.io/workloadselector: apps.deployment-car-serv-car-serv-deployment
      namespace: car-serv
    spec:
      containers:
        - image: docker.io/elroydevops/car-serv:latest
          imagePullPolicy: Always
          name: car-serv
          ports:
            - containerPort: 80
              name: tcp
              protocol: TCP
```

- Bây giờ có thể xóa deployment vừa rồi và import fiel yml trên vào vẫn sẽ ra kết quả như ban đầu

![](./images/4.png)

![](./images/5.png)

![](./images/6.png)
