- Cài đặt openssh-server

``` sh
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

- Cấu hình ip
  - Chú ý sẽ có 2 kiểu netwwork 2 kiểu này sẽ thay đổi cấu hình gateway4:
  - Chú ý sẽ có 2 kiểu netwwork 2 kiểu này sẽ thay đổi cấu hình gateway4:
    - Brige: Là ip cùng mạng LAN, gateway4: 192.168.1.1
    - NAT: Nội bộ máy local, gateway4 là ip vào Edit>Virtual Network Editer tìm dòng NAT NAT Setting>Gateway Ip

``` sh
# Chỉnh sửa file config netplan
nano /etc/netplan/00-installer-config.yaml
```

![Netplan](./netplan.png)

- Ctrl+x y để thoát và lưu

``` sh
# Apply cònig
netplan apply
# kiểm tra xem ở phần es330 ăn ip chưa
ip a
reboot
```
