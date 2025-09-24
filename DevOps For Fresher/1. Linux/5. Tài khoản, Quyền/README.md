# Tài khoản & Quyền

## 1. tài khoản

- tạo user

``` sh
useradd haitc1
```

adduser đầy đủ thông tin hơn

``` sh
adduser haitc2
```

- Chuyển user

``` sh
su haitc1
```

- Xem thông tin các user

``` sh
vim /etx/paswd
```

- Xóa user

``` sh
deluser haitc1
```

- tạo group

``` sh
groupadd devopt1
```

- Xóa group

``` sh
delgroup devopt2
```

- Thêm user vào group
  - a: append
  - G: Liệt kê các gr của user nếu không có -G sẽ xóa đi các gr hiện có

``` sh
usermod -aG <tên gr> <tên user>
```

- Kiểm tra user đang trong các group nào, lưu ý mặc định khi tạo user Linux tự động tạo 1 gr cùng tên

``` sh
groups haitc2
```

- Xóa user khỏi group

``` sh
deluser haitc2 devopt1
```

## 2, Quyền

- 1 thư mục có chủ sở hưu và nhóm sở hữu

``` sh
mkdir data
ls -l
# drwxr-xr-x 2 root root 4096 Jun 21 09:26 data
```

Ở đây root đầu tiên là chủ sở hữu, root thứ 2 là nhóm sở hữu.

- Chuyển chủ sở hữu, nhóm sở hữu
  - R: Dể thêm cả các thư mục, file con.

``` sh
chown -R root:devopt1 data
```

- Quyền truy cập
  - Khi ls -l thì sẽ có thông tin sau:
    - d: Là thư mục, file -
    - r: Quyền Read
    - w: Write
    - x: Execute
    - 3 ký tự tiếp là quyền của nhóm sở hữu
    - 3 Ký tự tiếp là quyền của các user khác
- Để thay đổi quyền thì cần có quyền x và có các từ khóa:
  - u là user
  - g là group
  - o là other

``` sh
# Thêm quyền đọc w ở thư mục data cho nhóm sở hữu
chmod g=rwx data/
# Chỉnh sửa nhiều quyền 1 lúc: u cho full, g cho r và other không cho làm gì
chmod u=rwx,g=r,o=- data/
```

- Có thể phân quyền bằng số:
  - r=4
  - w=2
  - x=1
    - Suy ra full quyển = 4+2+1 = 7

``` sh
# hai câu sau giống nhau
chmod -R u=rwx,g=rx,o=- data/
chmod -R 750 data/
```

# Cấu trúc thông tin hiển thị ls -l

- d: Phân biệt thư mục và file, file là dấu -
- 3 ký tự đầu là quyền người sở hữu
- 3 ký tự đầu là quyền nhóm sở hữu
- Số liên kết
- Tên người sở hữu
- Tên nhóm sở hữu
- Last mofiy date
- tên file/folder.
