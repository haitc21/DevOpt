# 1. Cài đặt docker

``` sh
mkdir -p /tools/docker
cd /tools/docker
vi nano install-docker.sh
```

- Nội dung file install-docker.sh

``` install-docker.sh
#!/bin/bash
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update -y
sudo apt install docker-ce -y
sudo systemctl start docker
sudo systemctl enable docker
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

- Chạy file install-docker.sh

``` sh
sh install-docker.sh
```

- Kiểm tra phiên bản

``` sh
docker --version
docker-compose --version
```

# 2. Các lệnh cơ bản

- Truy cập [hub docker](https://hub.docker.com/) để tìm image
- Pull image về máy
  - Tên image không có gì mặc đinh là bản mới nhất latest
  - Sau dấu : là tag phiên bản

``` sh
docker pull ubuntu
docker pull ubuntu:oracular-20240811.1
```

- Chạy image tạo container
  - --name: Tên container
  - -it: Truy cập luôn vào container
  - -d: chạy ẩn
  - -p: cấu hình port <pport máy chạy docker>:<port trong container> VD: map cổng 8888 của máy với 80 của container thì -p 8888:80

``` sh
docker un --name ubuntu -it ubuntu:oracular-20240811.1
```

- Xem các container

``` sh
# các container đang chạy
docker ps
# toàn bộ container
docker ps -a
```

- Khởi động container

``` sh
docker start ubuntu
```

- Truy cập vào docker đang chạy: docker exec -it <container_id_or_name> <lệnh cần chạy>

``` sh
docker exec -it ubuntu bash
```

- Xóa container: docker rm -f <container_id_or_name>

``` sh
docker rm -f ubuntu
# Xóa toàn bộ
docker rm -f $(docker ps -a)
```

- Xóa docker image: docker rmi <image_id_or_name>

``` sh
docker rmi ubuntu:oracular-20240811.1
```

- docker image prune: xóa tất cả image dangling (không còn tag hoặc không còn container nào dùng).
- docker image prune -a: xóa luôn tất cả image không được container nào dùng, kể cả có tag.
- docker system prune: dọn rộng hơn – xóa container đã dừng, network không dùng, dangling images và có thể cả volume (nếu thêm --volumes).
