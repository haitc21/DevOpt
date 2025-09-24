- Phiên bản OS

``` sh
cat /etc/os-release
```

- Kiểm tra Ram

``` sh
free -m
```

- Kiểm tra disk

``` sh
df -h /
```

- Kiểm tra tiến trình giống task Manager

``` sh
top
```

- Đổi tên server (host name)

``` sh
udo hostnamectl set-hostname lab-server
```

- Khởi động lại server

``` sh
sudo reboot
```

- Luôn thực hiện câu lệnh ở quyền Root
``` sh
sudo -i
```

- Thông tin kết nối
    - t: Thông tin kết nối TCP.
    - l: Các cổng đang mở và lắng nghe kết nối
    - p: Hiển thị tiến trình và chương trình liên quan đến kết nối.
    - u: thông tin kết nối UDP.
    - n: Hiển thị địa chỉ ip và cổng theo dạng số chứ không phải theo tên.

``` sh
sudo apt install net-tools
netstat -tlpun
```

- Xem các tiến trình đang chạy trên hệ thống

``` sh
ps -ef
# tìm kiếm tiến trình có tên java
ps -ef | prep java
```

- Kiểm tra xem có mạng internet không (8.8.8.8 là ip của Google)

``` sh
ping 8.8.8.8
```

- Kiểm tra xem có thông kết nối đến server:port khác không
  - Cách 1: telnet

``` sh
telnet 192.168.1.119:80
```

    - Cách 2: traceroute

``` sh
# cài đặt traceroute
apt install traceroute -y
# kiểm tra kết nối -T: TCP, -p: port
traceroute -T -p 80 192.168.1.119
```

- Cài đặt

``` sh
# -y để đỡ phải ấn yes trong khi cài đặt
apt install <tên công cụ>
# gỡ cài đặt
apt remove <tên công cụ>
```
