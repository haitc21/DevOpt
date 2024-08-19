# Cài đặt GitLab

[Video hướng dẫn](https://www.youtube.com/watch?v=Q6C5CoUxMuQ&list=PLsvroIvFNP1KU8foUeCC-hbJbqnAggWL2&index=13)

1. Lên gg search "gitlab ee package"
2. Tìm phiên bản phù hợp hệ điều hành Distro/Version
3. Chạy câu lệnh cài đặt, ở đây cài bản 17

- Tải package

``` sh
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
```

- Install

``` sh
sudo apt-get install gitlab-ee=17.3.0-ee.0
```

4. Cấu hình domain

- Vì không có domain nên đành add host
- Sửa file host trên gitlab-server

``` sh
sudo vim /etc/host
```

ấn "i" để vào insert, nhập "192.168.1.121 gitlab.haitc.local"

- Sửa file cấu hình gitlab, sửa url giống với external_url trong file host mới thêm <http://gitlab.haitc.local>.

``` sh
sudo vim /etc/gitlab/gitlab.rb
```

- Cập nhật config

``` sh
gitlab-ctl reconfigure
```

- Sửa file host trên window: c:\Windows\System32\Drivers\etc\hosts
 192.168.1.121 gitlab.haitc.local

5. Truy cập gitlab với domain vừa tạo ở trên

- Tài khoản: root
- Mật khẩu mặc định lấy trong

``` sh
cat /etc/gitlab/initial_root_password
```

4. Đổi mật khẩu root
