- Thư mục hiện tại

``` sh
pưd 
```

- tài khoản hiện tại

``` sh
whoami
```

- Danh sách thư mục

``` sh
ls
# Danh sách thư mục khác
la /var/
# hiển thị tất cả
ls -a
# hiển thị dưới dạng list
ls -l
# Sắp xếp theo thời gian tạo
ls -t
# kết hợp các opt
ls -alt
```

- Tạo thư mục

```  sh
mkdir dât
# tạo thư mục con
mkdir -p data1/data11
```

- Tạo file

``` sh
touch test.txt
```

- Copy

``` sh
cp data1/data11 data
# Copy cả file
cp -r data1/data11 data
# copy file rồi đổi tên
cp -r data1/data11/text.txt data/text2.txt
```

- Xóa

``` sh
# Xóa thư mục
rm data
# Xóa cả file
rm -r data
```

- Di chuyển

``` sh
mv -r data1/data11 data
```

- Lệnh echo: Dùng để in ra màn hình, in thông tin biến môi trường, ghi thông tin vào file

``` sh
echo hello > test3.txt
# Thêm txxt vào dòng tiếp theo
echo tranHai >> test3.txt
```

- Đọc thông tin file

``` sh
cat test3.txt
# đọc một số dòng cuối 
tail -n 1 test3.txt
# Đọc và lắng nghe thay đổi file như lúc xem log runtime
tail -f test3.txt
```
