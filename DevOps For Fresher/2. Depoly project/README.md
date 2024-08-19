# Triển khai dữ án

- Công cụ tương tứng với dự án.
- Chỉ cần chú ý đến file cấu hình.
- Triển khai đều chỉ có 2 bước: 1 Build, 2 Run

# Lưu ý

- Mỗi project thì có user tương ứng được quyền làm việc. Nên đặt cùng tên với dự án.
- Mỗi project có thư mục riêng.
- Không sử dụng user root để deploy.

# 1. Front-end

- [Triển khai VueJs, React sử dụng 3 cách chạy: Trực tiếp bằng npm server, Nginx, chạy như 1 service](https://www.youtube.com/watch?v=uqNfJj6msjA&list=PLsvroIvFNP1KU8foUeCC-hbJbqnAggWL2&index=10)

### Bước 1: Copy source lên server, tạo thư mục, tạo user, group, phân quyền

``` sh
# copy file
scp todolist.zip haitc@192.168.19.110:/home/haitc
# giải nén, tạo thư mục
sudo -i
apt install unzip
cd /haitc
unzip todolist
# tạo thư mục dự án
mkdir projects
mv todolist projects
# tạo user cùng tên dự án
adduser todolist
# thay đổi quyền sở hữu
chown -R todolist:todolist /projects/todolist
# Phân lại quyền
chmod -R 750 /projects/todolist
```

### Bước 2: Cài đặt môi trường

- nvm

``` sh
sudo apt install curl
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.bashrc
```

- nodejs

``` sh
nvm install --lts
```

### Nước 3: Build dữ án

``` sh
cd /projects/todolist
# chuyển user
su todolist
npm i
# xem cấu hình dự án Vue
vi vue.config.js
# build xong sẽ ra thư mục dist
npm run build
```

### Bước 4: Deploy

- Có các cách sau để deploy:
  - Run trực tiếp bằng vpm.
  - Web server:  Nginx, httpd
  - Chạy như 1 service của OS.
  - PM2 (cách này không được trình bày).

#### Cách 1: Run trực tiếp

``` sh
# chạy dự án theo cấu hình ở vue.config.js thì ở cổng 3000
npm run serve
```

#### Cách 2: Dùng Web Server ở đây dùng Nginx

``` sh
# cài ngĩnx mặc định chạy cổng 80 với http và 443 với https
apt install nginx
# kiểm tra port: tất cả nơi nào thông đến server thì sẽ hiển thị 0.0.0.0
netstat -tlpun
# kiểm tra cổng mặc định 
vi size-available/default
# test nginx
nginx -t
# restart nginx
systemctl restart nginx
# tạo file config cho website
vi conf.d/todolist.conf
# nginx cũng có 1 user mặc định là www-data, để kiểm tra vào file
vi /ect/nginx/nginx.conf
# Để dự án chạy được trong nginx thì user www-data phải ở trong group todolist
useradd -aG todolist www-data
systemctl restart nginx
```

nội dung file todolist.conf, cấu hình chạy ví dụ ở cổng 8999

```
server {
    listen 8999;
    root /projects/todolist/dist;
    index index.html;
    try_file $uri $uri/ /index.html;
}
```

#### Cách 3: Chạy như 1 Service hệ thống

``` sh
vi /lib/systemd/system/todolist.service
```

```
[Service]
Type=simple
User=todolist
Restart=on-failure
WorkingDirectory=/projects/todolist
ExecStart=npm run serve
```

``` sh
systemctl demon-reload
systemctl start todolist
systemctl status todolist
```

``` SH
# không nên sử dụng câu systemctl restart nginx vì nó sẽ ảnh hưởng đến các web khác cùng chạy tỏng nginx. Hãy dùng reload
nginx -f reload
```

# Back-end

- [Triển khai BE java, MariaDB (Mýql)](https://www.youtube.com/watch?v=qATtJZxSo7o&list=PLsvroIvFNP1KU8foUeCC-hbJbqnAggWL2&index=12)

### Bước 1: Copy source lên server, tạo thư mục, tạo user, group, phân quyền

``` sh
# copy file
scp ecommerce.zip haitc@192.168.19.110:/home
```

- Trên Server

``` sh
sudo -i
# giải nén
unzip ecommerce.zip
# di chuyển vào thư mục projects
mv ecommerce /projects
# tạo user
adduser ecommerce
# Thay đổi quyền thư mục
chown -R ecommerce ./projects/ecommerce
# Thay đổi quyền
chmod -R 750 ./projects/ecommerce
# kiểm tra quyền
ls -l ./projects
```

### Bước 2: Cài đặt môi trường (Java, Meven)

- Vào file pom.xml để xem version Java cần cài đặt ví dụ cài java 17

``` sh
# Java
apt install openjdk-17-jdk openjdk-17-jre
java --version
# Maven
apt install maven
mvn -v
```

- Vào file application.properties hoặc application.yml trong src/main/resource để xem cấu hình DB
- Cài đặt Maria DB

``` sh
apt install mariadb-server
# kiểm tra port: tất cả nơi nào thông đến server thì sẽ hiển thị 0.0.0.0
# cần sửa config để public port 3306 của db
netstat -tlpun
# tặt db
systemctl stop mariadb
# sửa cấu hình
ls ./etc/mysql/mariadb.conf.d/
vi ./etc/mysql/mariadb.conf.d/50-server.cnf
# Sửa bind-address thành 0.0.0.0 để mạng nội bộ có thể connect
# khởi động lại db
systemctl restart mariadb
# kiểm tra lại
netstat -tlpun
```

- đăng nhập db

``` sh
mysql -u root
# xem db
show databaws;
# tạo db
create database ecommerce;
# tạo user trong db tên đăng nhập ecommerce mật khẩu 123456 (giống như trong file appsetting.properties)
create user 'ecommerce-admin'@'%' identified by '123456';
# gán quyền
grant all privilageson ecommerce.* to 'ecommerce-admin'@'%';
# lưu lại quyền
flush privilages;
```

- Đăng nhập thử user bằng máy local

``` sh
mysql -h 192.168.19.110 -P 3306 -u ecommerce-admin -p 123456
show database;
use ecommerce;
show tables;
# chạy sql
source /projects/ecommerce/init_db.sql;
show tables;
```

- Sửa lại appsetting.properties

### Bước 3: Build và run

``` sh
# đổi tài khoản sang tài khoản của dự án
su ecommerce
# cài đặt dependencies bỏ qua test, kết quả trong thư mục target
mvn clean install -DskipTest=true
# Run
java -jar target/ecommerce.jar 
# Chạy nền: không giữ terminal, sẽ ra 1 file nohup.out
nohup java -jar target/ecommerce.jar  2>&1 &
# tìm process
ps -rf | grep ecommerce
# Tắt tiến trình
kill -9 <id process>
```
