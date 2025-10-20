1. Cài đặt

- Tạo thư mục

``` sh
mkdir -p /tools/jenkins
cd /tools/jenkins
```

- Tạo file install.sh

``` sh
vi install.sh
```

``` install.sh
#!/bin/bash

sudo apt install fontconfig openjdk-17-jre -y
java --version
wget -p -O - https://pkg.jenkins.io/debian/jenkins.io.key | apt-key add -
sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5BA31D57EF5975CA
apt-get update
apt install jenkins -y
systemctl start jenkins
ufw allow 8080
```

- Run install.sh

``` sh
bash install.sh
```

- Kiểm tra

``` sh
systemctl status jenkins
```

2. Add Host
192.168.1.105 jenkins.haitc.local
port 8080
3. Lấy password mặc định

``` sh
cat /var/lib/jenkins/secrets/initialAdminPassword
```

4. Reverse proxy chuyển từ 8080 sang 80

- thêm jenkins.haitc.local vào /etc/hosts

``` sh
sudo apt install nginx -y
cd /etc/nginx/conf.d/
sudo vi jenkins.haitc.local.conf
```

``` jenkins.haitc.local.conf
server {
    listen 80;
    server_name jenkins.haitc.local;

    location / {
        proxy_pass http://jenkins.haitc.local:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

- Kiểm tra cấu hình và restart nginx

``` sh
sudo nginx -t
systemctl restart nginx
```

5. Các thành phần cơ bản của Jenkins
5.1 Manage Jenkins

- System
  - Home directory: /var/lib/jenkins
- Plugin
- Node
  - thay vì ssh thì sử dụng Jenkins Agent để kết nối
- Security: mặc định Jenkins không có phân quyền mà phải dùng plugin
- Credential: Lưu các bảo mật, tài khoản ....
- User
- System log
- Jenkins CLI: Tự động hóa mọi thứ
- Appearance: Cài đặt hiển thị (Dark mode...)
