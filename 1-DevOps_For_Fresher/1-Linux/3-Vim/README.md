- Cài đặt

``` sh
sudo apt install vim -y
```

- Mở hoặc tạo file mới

``` sh
vim text.txt
```

- trong vim có 2 mode là Insert và Command.
- Để thêm dữ liệu ấn phím "i" để vào Insert.
- Ấn "Esc" để thoát Insert

- Ở Command mode
  - "u" giống Ctrl+z
  - Ấn "dd" để xóa xòng
  - "yy" để copy dòng hiện tại, "pp" để paste
  - "/" để tìm kiếm, n để đến kết quả tiếp, N để quay lại, Enter để chuyển con trỏ đến kết quả
  - Gõ ":x" để lưu
  - Gõ ":q!" để thoát và không lưu
- gg di chuyển con trỏ đến dòng đầu tiên.
- V bắt đầu chế độ Visual Line.
- G di chuyển con trỏ đến dòng cuối cùng, chọn toàn bộ nội dung của tệp.
- Sau khi chọn toàn bộ nội dung bằng lệnh ggVG, gõ d để xóa tất cả nội dung đã chọn.
