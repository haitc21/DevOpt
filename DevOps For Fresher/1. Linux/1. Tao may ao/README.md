# Tạo máy ảo Ubunto bằng VMware Workstation Pro

1. Vào file > New Virtual Mechine
![](./image/s1.png)
2. Next
![](./image/s2.png)
3. Chọn file iso
![](./image/s3.png)
4. Điền tk/mk
![](./image/s4.png)
5. Đặt tên máy ảo và nơi lưu trữ
![](./image/s5.png)
6. Đặt giới hạn dung lượng
![](./image/s7.png)
7. Cấu hình phần cứng

- Customize Hardware
![](./image/s7-1.png)
- Giới hạn RAM
![](./image/s7-2.png)
- Cấu hình Processor
![](./image/s7-3.png)
- Network đổi về Brige (mặc định là NAT)

![](./image/s7-4.png)

- Chú ý sẽ có 2 kiểu netwwork 2 kiểu này sẽ thay đổi cấu hình gateway4:
  - Brige: Là ip cùng mạng LAN, gateway4: 192.168.1.1
  - NAT: Nội bộ máy local, gateway4 là ip vào Edit>Virtual Network Editer tìm dòng NAT NAT Setting>Gateway Ip

8. Close > Finish
9. Bật VM vừa tạo
![](./image/s9.png)
10. Cấu hình VM

- Chọn ngôn ngữ English > Enter
![](./image/s10-1.png)
- Không cần update > Enter

![](./image/s10-2.png)

- Keyboard
![](./image/s10-3.png)

- Network: enS3 là tên card mạng, ip ở dưới là ip động sẽ thay đổi khi restart VM nên sau đó cần cấu hình lại
![](./image/s10-4.png)

- Proxy để mặc định
![](./image/s10-5.png)

- Miror để mặc định
![](./image/s10-6.png)

- Storage để mặc định
![](./image/s10-7.png)
- Disk để dung lượng dòng Ubuntu-LV bằng dòng trên
![](./image/s10-8.png)
- Điền thông tin
![](./image/s10-9.png)
- Cài đặt OpenSSH nhớ tích vào không là phải cài tay như sau:

``` sh
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

![](./image/s10-10.png)

- Cần cài thêm gì thì tick vào
![](./image/s10-11.png)
- Chờ cho VM chạy xong
![](./image/s10-12.png)

11. Cấu hình ip

- Chỉnh sửa file config netplan

``` sh
sudo nano /etc/netplan/00-installer-config.yaml
```

![Netplan](./image/netplan.png)

- Ctrl+x y để thoát và lưu

``` sh
# Apply cònig
netplan apply
# kiểm tra xem ở phần es330 ăn ip chưa
ip a
# restart server
reboot
```
