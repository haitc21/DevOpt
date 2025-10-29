# Backup

- Backup ứng dụng:
  - Khi triển khai dự án chúngta đã sử dụng PV & PVC lưu tữ ở nfs-server. Chúng ta có thể chạy **cron-task** để backup tự động.
  - Thông thường trên on-prem thì nên triển khai công cụ replica (1 master, 2 replica) và có thêm 1 backup ở bên ngoài
- Backup manifét: Là backup các file cấu hình yaml,jsn... có thể dùng **GitOps**.
- Backup cấu hình dự án: Backup **ConfigMap**, **Secret** cần lưu trữ bảo mật nên dùng các giải pháp **KMS - Key Management Service** như **HashiCorp Vault**.
- Backup trạng thái K8s Cluster: **etcd**.
- Backup cluster metadata: Ansible, Terraform.
- Backup cấu hình monitoring & logging.
- Backup cấu hình công cụ.

# Backup & Restore K8s Cluster

- Công cụ sử dụng [**Velero**](https://velero.io/).
- Velero cần công cụ lưu trữ s3, trên on-prem sử dụng **Minio**.
- Cấu hình docker-compose.yml cài đặt MiniO (thực hiện trên sv5)

```yml
version: '3'
services:
  minio:
    image: minio/minio:RELEASE.2023-01-12T02-06-16Z
    container_name: minio
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - ./storage:/data
    environment:
      MINIO_ROOT_USER: devopseduvn
      MINIO_ROOT_PASSWORD: devopseduvn
    command: server --console-address ":9001" /data
 ```

- Truy cập <http://192.168.159.105:9001/login> tài khoản devopseduvn/devopseduvn
- Tạo bucket `k8s-devopseduvn-cluster-backup`
- tạo Access key ví dụ
  - Access Key: `x4njfRrLjGNc8XGr`
  - Secret Key: `aye2wx8csUGFBqyPBxoqCFqASCUJbE6o`

- Google search `Velero Release` rồi tìm bản phù hợp lấy link tải về.
- Cài đặt Velero client (Thực hiện trên server sv1 hoặc kubectl shell Rancher)

```sh
wget https://github.com/vmware-tanzu/velero/releases/download/v1.15.0/velero-v1.15.0-linux-amd64.tar.gz
tar -xvf velero-v1.15.0-linux-amd64.tar.gz
sudo mv velero-v1.15.0-linux-amd64/velero /usr/local/bin
```

- Cấu hình các biến MiniO (thực hiện trên server sv1 hoặc kubectl shell Rancher)

```sh
export MINIO_URL="http://192.168.159.105:9000"
export MINIO_ACCESS_KEY_ID="x4njfRrLjGNc8XGr"
export MINIO_SECRET_KEY_ID="aye2wx8csUGFBqyPBxoqCFqASCUJbE6o"
export MINIO_BUCKET="k8s-devopseduvn-cluster-backup"
```

- Cài đặt Velero với MinIO làm backend lưu trữ các bản sao lưu của Kubernetes (thực hiện trên server sv1 hoặc kubectl shell Rancher)

```sh
velero install \
  --provider aws \
  --bucket $MINIO_BUCKET \
  --secret-file <(echo -e "[default]naws_access_key_id=$MINIO_ACCESS_KEY_IDnaws_secret_access_key=$MINIO_SECRET_KEY_ID") \
  --use-node-agent \
  --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=$MINIO_URL \
  --plugins velero/velero-plugin-for-aws:v1.5.0 \
  --namespace velero
```

- Các tài nguyên sẽ đượ khởi tạo trên k8s clusster tỏng  namespace `velero`.

>Note: Kiểm tra các tài nguyên velero khởi tạo xong hết mới làm bước tiếp theo.

- Backup tài nguyên của Namespace cụ thể (thực hiện trên server sv1 hoặc kubectl shell Rancher)

```sh
velero backup create ecommerce-v1 --include-namespaces ecommerce
velero backup get
```

- Restore bản backup cụ thể (thực hiện trên server sv1 hoặc kubectl shell Rancher)

```sh
velero restore create ecommerce-v1 --from-backup ecommerce-v1 --include-namespace=ecommerce
```

- Đặt lịch tự động backup – ví dụ backup hàng ngày (thực hiện trên server sv1 hoặc kubectl shell Rancher)

```sh
velero schedule create daily-cluster-backup --schedule="0 0 ** *" --include-namespaces '*'
```