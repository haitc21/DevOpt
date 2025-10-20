# Rancher

 Rancher là 1 công cụ mã nguồn mở, chạy nền web, hỗ trợ triển khai và quản lý nhiều cụm kubernetes cả trên on-prem và cloud

## 1. Tạo server

- tạo máy ảo sv4 với ip: 192.168.159.104
- **Tạo hard disk lưu dữ liệu rancher**: VMware => VM Settings => Add => Hard Disk
- truy cập sv4 kiểm tra xem hard disk đã nhận chưa

````sh
lsblk
```

- Có dòng `sdb                         8:16   0   10G  0 disk` là ok.

- Gán disk (ổ cứng) vào trong thư mục /data

``` sh
sudo mkfs.ext4 -m 0 /dev/sdb
mkdir /data
echo "/dev/sdb  /data  ext4  defaults  0  0" | sudo tee -a /etc/fstab
mount -a
sudo df -h
```

## 2. Cài đặt rancher

Có thể cài đặt rancher qua Docker hoặc k8s. Ở đây để đơn giản sẽ cài đặt qua Docker

- Tạo 1 server tên là **rancher** trên VMwware với ip 192.168.159.104
- Add host cho các server cài cụm k8s
- [Cài đặt Docker](../../DevOps%20For%20Fresher/5.%20Docker/1%20Cài%20đặt%20docker.md)
- Tạo thư mục chứa rancher

>Note: Cần kiểm tra phiên bản rancher có tương thích với phiên bản k8s hiện tại không bằng cách tìm **rancher version metrix**. Ví dụ ở bài trước k8s được cài là phiên bản 1.30.14 (ở sv1 chạy kubectl get no thì sẽ hiện cột VERSION) thì ở đây cài rancher v2.9.2

````sh
mkdir -p /data/rancher
cd /data/rancher/
vi docker-compose.yml
```

- File docker-compose.yml

```yml
version: '3'
services:
  rancher-server:
    image: rancher/rancher:v2.10.4-alpha3
    container_name: rancher-server
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /data/rancher/data:/var/lib/rancher
    privileged: true
```

- Khởi chạy rancher

````sh
docker-compose up -d
```

- Truy cập vào [reancher](https://rancher.local)
- Lấy mật khẩu user **admin**

````sh
docker logs rancher-server 2>&1 | grep "Bootstrap Password:"
```

- Đổi mật khẩu

## 2. Kết nối rancher đến cụm k8s

- Trên [reancher](https://rancher.local) nhấn **Import Exsisting**
- Generic
- Đặt Cluster Name, ở đây là **devopseduvn**
- Create
- Vì rancher dùng SSL/TLS tự ký nên sẽ kết nổi bằng lệnh ở dòng **If you get a "certificate signed by unknown authority" error, your Rancher installation has a self-signed or untrusted SSL certificate. Run the command below instead to bypass the certificate verification:**. Chạy lệnh ở sv1 (master node)

````sh
curl --insecure -sfL https://rancher.local/v3/import/m6h2glbrrwwnffqsmxg5f5nnk8jsxrv6rdk4lpdzpt42wxsxb2hgmb_c-m-p5bm2cfb.yaml | kubectl apply -f -
```
