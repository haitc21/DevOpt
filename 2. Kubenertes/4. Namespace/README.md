# Namespace trong k8s

- Namespace là một cách tổ chức và phân tách các tài nguyên trong 1 cụm k8s giúp quản lý tốt hơn.
- Chia nhỏ các tài nguyên của 1 cụm lớn thành các không gian logic nhỏ hơn. Việc này giúp tách riêng các không gian làm việc và có thể thực thi các chính sách như hạn chế tài nguyên cpu mem, phân quyền người truy cập
- Ví dụ: Công ty có 1 cụm k8s có 3 dự án A,B, C và mỗi dự án có 3 môi trường develop, stagging, production thì có thể chia thành các namspace: projaect-a-dev, proect-a-stagging, project-a-prod, projaect-b-dev...
- Mặc định khi tạo cụm k8s thì sẽ có namespace mặc định là **default**

````sh
kubectl get pod
kubectl get pod --namespace default
```

- Lấy danh sách namespace

``` sh
kubectl get namepsace
kubectl get ns
```

>Note: Trong lệnh kubectl thì namespace có thể viết tắt là ns

- Tạo namespace tên là project-1

````sh
kubectl create ns project-1
```

- Xóa namespace

````sh
kubectl delete ns project-1
```

- Thông thường thì sẽ không dùng lệnh tạo namespace trực tiếp mà sử dụng file yml

``` sh
mkdir -p projects/project-1
cd -p projects/project-1
vi ns.yml
```

- Nội dung file ns.yml

```yml
apiVersion: v1
kind: Namespace
metadata:
  name: project-1
```

- Appluy cấu hình

````sh
kubectl apply -f ns.yml
```

>Note: -f là chỉ định file

- Xóa namespace

````sh
kubectl delete -f ns.yml
```

- Giới hạn tài nguyên trong namespace

````sh
vi resourcequota.yml
```

- Nội dung file resourcequota.yml

```yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-quota
  namespace: project-1
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
```
