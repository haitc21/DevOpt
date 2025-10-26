# TTriển khai Redis trên k8s

B
bài này sẽ sử dụng **helm** để triển khai Redis trên k8s

- trên **sv1**

```sh
kubectl create namespace architecture
mkdir redis && cd redis
```

- Lên rancher tạo project `architecture` và `move` namespace `architecture` vào project.

- tạo thư mục redis trên `nfs-serrver`(**sv5**)

```sh
cd /data
mkidir redis
sudo chown -R nobody:nogroup /data/redis
sudo chmod -R 777 /data/redis
```

- tạo PV và PVC

```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-pv
spec:
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: /data/redis/
    server: 192.168.159.105 #Chú ý thay địa chỉ IP tương ứng của bạn
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs-storage
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis-pvc
  namespace: architecture
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 8Gi
  storageClassName: nfs-storage
```

- Google search `redis chart`, có thể thấy github của `bitnami` cho ![Redis chart](https://github.com/bitnami/charts/tree/main/bitnami/redis).
- Cấu hình chart chỉ cần tạo file **values.yaml** sẽ ghi đè cấu hình trong file **values.yaml** gốc của Chart.
- Cấu hình file **values.yaml** của Redis chart (thực hiện trên server sv1 hoặc kubectl shell Rancher)

```sh
vi values.yaml
```

```yaml
architecture: replication

auth:
architecture: replication

auth:
  enabled: true
  password: "devopseduvn"

master:
  persistence:
    enabled: true
    existingClaim: redis-pvc
    size: 2Gi

replica:
  replicaCount: 3
  persistence:
    enabled: true
    existingClaim: redis-pvc
    size: 2Gi

sentinel:
  enabled: true
  replicas: 3
 ```

 >Note: Luôn có **enabled: true**

- Khởi tạo Redis Chart (thực hiện trên server sv1 hoặc kubectl shell Rancher)

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install redis-sentinel bitnami/redis --values values.yaml --namespace architecture
```

- Kết nối thử dự án ecommerce backend vào trong Redis và kiểm tra hoạt động (thực hiện trong Pod ecommerce backend)

```sh
apk update
apk add redis
# host: <service-name>.<namespace>.svc.cluster.local
redis-cli -h redis-sentinel.architecture.svc.cluster.local -p 26379 -a devopseduvn
# Redis cli
# lấy địa chỉ để kết nối vào reddis, ở trên mới chỉ là sentinel
SENTINEL get-master-addr-by-name mymaster
exit
# kết nối vòa redis master
redis-cli -h redis-sentinel-node-0.redis-sentinel-headless.architecture.svc.cluster.local -p 6379 -a devopseduvn
SET k8s_course "K8s series by devopseduvn"
GET k8s_course
```

- Lên **sv4** để kiểm tra các file lưu trữ

```sh
haitc@sv5:/data$ ls /data/redis/
appendonlydir  dump.rdb
haitc@sv5:/data$ ls /data/redis/appendonlydir
appendonly.aof.3.base.rdb  appendonly.aof.3.incr.aof  appendonly.aof.manifest
```

- `appendonly.aof.3.base.rdb`: File snapshot lưu trữ trạng thái cảu redis trong thười điểm cụ thể.
- `appendonly.aof.3.incr.aof`: Ghi lại các lệnh đọc ghi vào redis. Khi redis khởi động thì sẽ thực hiện các lệnh này để khôi phục dữ liệu.
- `appendonly.aof.manifest`: Chứa thông tin các file `aof` để redis có thể đọc và theo dõi trạng thái.
