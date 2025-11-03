# HCL Syntax

## 1. Terraform Configuration Syntax

Trang này đi sâu hơn vào **cú pháp cấp thấp (low-level syntax)**, mô tả các **thành phần cơ bản (building blocks)** mà Terraform sử dụng để xây dựng ngôn ngữ của mình.

Terraform sử dụng cú pháp **native syntax**
Ngoài ra, các cấu trúc của Terraform cũng có thể được **biểu diễn bằng cú pháp JSON**, thuận tiện cho việc **xử lý bằng lập trình**, nhưng **khó đọc hơn** đối với con người
---

### 1.1. Cấu trúc cú pháp chính: **Arguments** và **Blocks**

#### 1.1.1. **Arguments**

**Argument** gán giá trị cho một tên cụ thể:

```hcl
image_id = "abc123"
```

- Phần trước dấu = → tên của argument (image_id)
- Phần sau dấu = → giá trị của argument ("abc123")

Ngữ cảnh nơi argument xuất hiện sẽ xác định kiểu dữ liệu hợp lệ.
Ví dụ: Mỗi loại resource có một schema quy định các kiểu dữ liệu được phép cho từng argument.
Tuy nhiên, nhiều argument chấp nhận các biểu thức (expressions), cho phép giá trị được tính toán tự động từ các giá trị khác.

 >Tips:Ngôn ngữ Terraform dựa trên HCL.
Trong tài liệu HCL, thuật ngữ "attribute" thường được dùng thay cho "argument".
Tuy nhiên, trong Terraform, từ attribute còn được dùng để chỉ các thuộc tính của tài nguyên (ví dụ id) — có thể được tham chiếu nhưng không thể gán trực tiếp.Vì vậy, Terraform sử dụng từ "argument" trong tài liệu để tránh nhầm lẫn.

#### 1.1.2. Blocks

```hcl
resource "aws_instance" "example" {
  ami = "abc123"

  network_interface {
    # ...
  }
}
```

Một block có:

- Loại (type): ở ví dụ trên là resource.
- Nhãn (labels): mỗi loại block xác định số lượng nhãn cần theo sau.
  - resource yêu cầu hai nhãn:
    - aws_instance: loại tài nguyên (đến từ AWS provider)
    - example: tên tùy ý để định danh instance này
- Bên trong phần thân, bạn có thể khai báo các argument và block lồng nhau, tạo nên cấu trúc phân cấp (hierarchy).

### 1.2. Identifiers (Định danh)

Các tên của argument, block type, và các khái niệm đặc thù trong Terraform (như `resources`, `variables`, v.v.) đều là identifiers.
Identifier có thể chứa:

- Chữ cái (letters)
- Chữ số (digits)
- Gạch dưới _
- Dấu gạch ngang -

>Note: Ký tự đầu tiên không được là chữ số, để tránh nhầm lẫn với số nguyên.
>
### 1.3. Comment

| Cú pháp     | Loại comment | Mô tả                          |
| ----------- | ------------ | ------------------------------ |
| `#`         | Một dòng     | Kết thúc ở cuối dòng           |
| `//`        | Một dòng     | Tương tự `#`, ít dùng hơn      |
| `/* ... */` | Nhiều dòng   | Có thể trải dài qua nhiều dòng |

### 1.4. Character Encoding và Line Endings

Tệp cấu hình Terraform phải được mã hóa **UTF-8**.

## 2. JSON

Cú pháp JSON không được sử dụng phổ biến. Có thể tiemf hiểu thêm [tại đây](https://developer.hashicorp.com/terraform/language/syntax/json)
