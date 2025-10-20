# Terraform Get Started AWS

Hướng dẫn này qua việc sử dụng Terraform để tạo máy ảo EC@ trên AWS sẽ giưới thiệu các khái niệm cơ bản vể Terraform

## 1. Chuẩn bị

- ![Teraform CLI](../1.%20Giới%20thiệu/README.md)
- ~[AWS CLI](https://docs.aws.amazon.co:m/cli/latest/userguide/getting-started-install.html): Terraform sẽ sử dụng các công cụ từ provider để quản lý các resource trên provider đó. Ở đây provider là AWS nên sẽ dùng AWS CLI.

- ![Tài khoản AWS](https://aws.amazon.com/free/) và ![AWS security credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/security-creds.html) để xác thực

## 2. Create

### 2.1 WWrite Configuration

- Tạo thư mục `learn-terraform-get-started-aws`
- File cấu hình là 1 file plan text viết bằng **HashiCorp's configuration language - HCL** định dạng `.tf` Tạo file `terraform.tf` trong thư mục learn-terraform-get-started-aws`

#### 2.1.1 Terraform block

- Chứa thông tin version của teraform và thông tin các tên,version của các provider

```terraform.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.92"
    }
  }

  required_version = ">= 1.2"
}
```

- Teraform gọi các API của provider để quản lý resource.
- Có thể tìm kiếm thông tin chi tiết về các cấu hình resource trên ![Terraform Registry](https://registry.terraform.io/?product_intent=terraform)

### 2.1.2 Configuration blocks

- Nơi chứa thông tin cấu hình resource
- Tạo firl `main.tf`

```main.tf
provider "aws" {
  region = "us-west-2"
}

data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }

  owners = ["099720109477"] # Canonical
}

resource "aws_instance" "app_server" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"

  tags = {
    Name = "learn-terraform"
  }
}
```

#### 2.1.2.1 Providers

- Cấu trúc: provider "<provider_nmae>" {}
- provider_nmae trong required_providers của file `teraform.tf`.
- Thông tin cấu hình chi tiết trên ![Terraform Registry](https://registry.terraform.io/?product_intent=terraform)
- Trong `main.tf` có thể khai báo nhiều provider block tương ứng với cá provider khai báo trong required_providers của file `teraform.tf`.
- Thong tin auth teraform sử dung cấu hình của AWS CLi nếu chưa cấu hình có thể sử dụng 2 biến môi trường `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY`

````sh
aws configure list
```

#### 2.1.2.2. Data sources

- **data** là một cách để lấy thông tin từ bên ngoài — ví dụ như từ AWS, Azure, GCP, hoặc thậm chí từ một file khác — mà không cần phải tự tạo resource mới.
- Vị dụ lấy thông tin
- Ví dụ trong `main.tf`

```main.tf
data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }

  owners = ["099720109477"] # Canonical
}
```

| **Thành phần** | **Giải thích chi tiết** |
|-----------------|--------------------------|
| `data "aws_ami" "ubuntu"` | Khai báo một **data source** có loại là `aws_ami` (lấy thông tin Amazon Machine Image từ AWS) và đặt tên là `ubuntu`. Data source này **không tạo mới tài nguyên**, chỉ **truy vấn dữ liệu có sẵn**. |
| `most_recent = true` | Yêu cầu Terraform lấy **phiên bản mới nhất** của AMI trong số các kết quả phù hợp với bộ lọc. Giúp đảm bảo luôn dùng image mới nhất mà không cần cập nhật thủ công. |
| `filter { ... }` | Định nghĩa **bộ lọc tìm kiếm AMI** dựa trên các tiêu chí cụ thể như tên, ID, trạng thái, v.v. |
| `name = "name"` | Thuộc tính của AMI mà ta muốn lọc theo. Ở đây là trường `name` trong metadata của AMI. |
| `values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]` | Mẫu tên AMI cần tìm. Dấu `*` biểu thị ký tự đại diện (wildcard), giúp lấy tất cả AMI bắt đầu bằng chuỗi trên. Ví dụ: `ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-20241001`. |
| `owners = ["099720109477"]` | Xác định **chủ sở hữu AMI** để tránh dùng AMI không chính thức. `099720109477` là **AWS account ID của Canonical (Ubuntu)** — đảm bảo lấy đúng image gốc từ Ubuntu. |
| `data.aws_ami.ubuntu.id` | Cách **tham chiếu ID của AMI** được tìm thấy. Có thể dùng giá trị này trong các resource khác (ví dụ khi tạo `aws_instance`). |
| Ví dụ sử dụng | ```hcl<br>resource "aws_instance" "web" {<br>  ami           = data.aws_ami.ubuntu.id<br>  instance_type = "t3.micro"<br>}<br>``` <br>→ Tạo một EC2 instance mới sử dụng AMI Ubuntu mới nhất được tìm từ data source ở trên. |

---

✅ **Tóm lại:**  
Data source `aws_ami.ubuntu` giúp Terraform **tự động tìm AMI mới nhất** của Ubuntu mà không cần **hardcode ID**, đảm bảo cấu hình **linh hoạt, cập nhật và an toàn**.

- Sau khi Terraform tải thông tin AMI từ AWS (khi chạy terraform plan hoặc terraform apply), bạn có thể tham chiếu tới dữ liệu này bằng cú pháp:

```hcl
data.aws_ami.ubuntu.<attribute_name>
```

- Ví dụ sử dụng trong resource

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
}
```

#### 2.1.2.3. Resources

- Là nơi khai báo các Resource để triển khai trên provider.
- Cấu trúc: resource`<resource type> <resource name> {}`
  - resource type>: Provider định nghĩa.
  - <resource name: Tên định danh nội bộ trong file Terraform,
- Trong rresource block có các tham số **arguments** do provider định nghĩa để cấu hình infrastructure.
- Ví dụ cấu hình AWS EC2

```main.tf
resource "aws_instance" "app_server" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"

  tags = {
    Name = "learn-terraform"
  }
}
```

| **Thành phần**                         | **Giải thích chi tiết**                                                                                                                                                                                                                                                                      |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `resource "aws_instance" "app_server"` | Khai báo một **resource block** để tạo một tài nguyên thật (ở đây là **EC2 instance** trên AWS). <br><br>- `aws_instance` là **loại tài nguyên** (resource type) được định nghĩa bởi **AWS provider**.<br>- `app_server` là **tên nội bộ** của resource, dùng để tham chiếu trong Terraform. |
| `ami = data.aws_ami.ubuntu.id`         | Xác định **Amazon Machine Image (AMI)** dùng để khởi tạo EC2 instance. <br>Giá trị được lấy từ **data source** `data.aws_ami.ubuntu`, đảm bảo luôn chọn bản Ubuntu mới nhất.                                                                                                                 |
| `instance_type = "t2.micro"`           | Định nghĩa **loại máy EC2** cần tạo. <br>`t2.micro` là loại nhỏ, nằm trong **AWS Free Tier**, phù hợp cho thử nghiệm và học tập.                                                                                                                                                             |
| `tags = { Name = "learn-terraform" }`  | Thêm **tag (thẻ)** cho instance. Tag giúp dễ dàng quản lý tài nguyên trên AWS. <br>Ở đây, tag `Name` được đặt là `"learn-terraform"`.                                                                                                                                                        |

## 2.2. Format configuration

- Dùng lệnh `terraform fmt` để reformat (bao gồm syntax và style code) tất cả các file tỏng thư mục

| **Tùy chọn**               | **Ý nghĩa**                                                                    |
| -------------------------- | ------------------------------------------------------------------------------ |
| `terraform fmt`            | Định dạng file trong thư mục hiện tại.                                         |
| `terraform fmt -recursive` | Định dạng tất cả các file trong **thư mục con** (rất hữu ích trong dự án lớn). |
| `terraform fmt -check`     | Kiểm tra xem có file nào **chưa đúng định dạng** hay không (không chỉnh sửa).  |

## 2.3. Initialize your workspace

- lần đầu tiên cần init để teraform khởi tạo

```hcl
terraform init
```

- Terraform sẽ:
  - Tải (download) provider cần thiết (ở đây là aws) từ Terraform Registry.
  - Cài đặt provider vào thư mục ẩn .terraform trong thư mục làm việc hiện tại.
  - Tạo file khóa phiên bản .terraform.lock.hcl.

### 2.4. Validate configuration

- Để kiểm tra syntax và đảm bảo các provider đã cài đặt thành công

````sh
terraform validate
```

### 2.5. Plan

- Xem excution plan trước khi apply

````sh
terraform plan
```

- Cách làm này giúp bạn **phát hiện lỗi cấu hình hoặc thay đổi ngoài ý muốn** trước khi Terraform tác động lên hạ tầng thật.

### 2.6. Create infrastructure

- Tạo cơ sở hạ tầng với Terraform `terraform apply`

- Terraform thực hiện việc **tạo hoặc thay đổi hạ tầng** theo **2 bước chính**:

  - **Lập kế hoạch thực thi (Execution Plan)**: Terraform phân tích file cấu hình và hiển thị **những thay đổi sẽ thực hiện**.
  - **Thực thi thay đổi (Apply)**: Sau khi bạn **xác nhận** (nhập `yes`), Terraform **thực hiện các thay đổi đó** thông qua provider tương ứng (ví dụ: AWS).

```bash
terraform apply
```

- Terraform sẽ:
  - Tải thông tin từ data source (aws_ami.ubuntu),
  - Tạo kế hoạch thực thi (plan),
  - Hiển thị chi tiết các thay đổi sắp thực hiện.
- Ví dụ đầu ra:

````sh
$ terraform apply
data.aws_ami.ubuntu: Reading...
data.aws_ami.ubuntu: Read complete after 1s [id=ami-0026a04369a3093cc]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.app_server will be created
  + resource "aws_instance" "app_server" {
      + ami                                  = "ami-0026a04369a3093cc"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = (known after apply)
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = (known after apply)
      + ebs_optimized                        = (known after apply)
      + enable_primary_ipv6                  = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + host_resource_group_arn              = (known after apply)
      + iam_instance_profile                 = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_lifecycle                   = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t2.micro"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = (known after apply)
      + monitoring                           = (known after apply)
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + spot_instance_request_id             = (known after apply)
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Name" = "learn-terraform"
        }
      + tags_all                             = {
          + "Name" = "learn-terraform"
        }
      + tenancy                              = (known after apply)
      + user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification (known after apply)

      + cpu_options (known after apply)

      + ebs_block_device (known after apply)

      + enclave_options (known after apply)

      + ephemeral_block_device (known after apply)

      + instance_market_options (known after apply)

      + maintenance_options (known after apply)

      + metadata_options (known after apply)

      + network_interface (known after apply)

      + private_dns_name_options (known after apply)

      + root_block_device (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.
```

- Giải thích:| **Ký hiệu / Thuật ngữ**                      | **Giải thích**                                                                                      |
| -------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `+ create`                                   | Terraform sẽ **tạo mới** tài nguyên.                                                                |
| `~ update`                                   | Terraform sẽ **cập nhật** tài nguyên hiện có.                                                       |
| `- destroy`                                  | Terraform sẽ **xóa** tài nguyên.                                                                    |
| `(known after apply)`                        | Giá trị này chỉ được biết **sau khi tài nguyên được tạo** (ví dụ: `instance_id`, `public_ip`, ...). |
| `Plan: 1 to add, 0 to change, 0 to destroy.` | Terraform dự kiến **tạo 1 resource mới**, **không sửa** và **không xóa** gì cả.                     |

- Xác nhận `yes` để teraform bặt đầu tạo infrastructure.
- Đợi quá trình tạo xong lên AWS kiểm tra xem EC@ đã tạo đúng chưa.

### 2.7. Inspect state

- Sau khi tạo infrastructure xong thì teraform sẽ lưu trọng thái vào file `terraform.tfstate`
- Lấy danh sách resource và data source tỏng workspace hiện tại

````sh
 terraform state list
 ```

- Xem chi tiết

 ````sh
 terraform show
 ```

- Mỗi khi chạy lệnh `terraform plan` hoặc `terraform apply` thì teraform sẽ sử dụng file
`terraform.tfstate` để so sánh và đưa ra thông tin thay đổi.
- Mặc định thì `terraform.tfstate` sẽ được tạo ở local.
- **nỏe**: `terraform.tfstate` có thể chứa **thông tin nhạy cảm** như pasword, credential để kết nối đến các provider nên cần **bảo mật**. Có thể dùng HCP hoặc phân quyền Git để quản lý quyền truy cập `terraform.tfstate`

#### Các file đã tạo trong phần 2

- main.tf

```main.tf
provider "aws" {
  region = "us-west-2"
}

data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }

  owners = ["099720109477"] # Canonical
}

resource "aws_instance" "app_server" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"

  tags = {
    Name = "learn-terraform"
  }
}
```

- terraform.tf

```terraform.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.92"
    }
  }

  required_version = ">= 1.2"
}
```

## 3. Manage infrastructure

Pử phần 2 chúng ta đã ạo được terraform workspace cơ bản gồm 2 file `main.tf` và `terraform.tf` và đã triển khai infrastructure EC2 trên AWS bằng terraform.
Ở phần 3 chúng ta sẽ sử dụng biến (`variable`), output value, module để teraform dynamic và clean hơn

### 3.1. Variables and outputs

#### 3.1.1 Input variables

- Tạo file `variables.tf`

```variables.tf
variable "instance_name" {
  description = "Value of the EC2 instance's Name tag."
  type        = string
  default     = "learn-terraform"
}

variable "instance_type" {
  description = "The EC2 instance's type."
  type        = string
  default     = "t2.micro"
}
```

- Sử dụng 2 biến trên trong `main.tf`

```main.tf
resource "aws_instance" "app_server" {
   ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  instance_type = var.instance_type

  tags = {
   Name = "learn-terraform"
   Name = var.instance_name
  }
}
```

- bạn có thể thấy thay gán trực tiếp các giá trị arguments (hardcode) thì có thể sử dụng biến trong file `variables.tf` bằng cú pháp `var.<tên biến>`.
- Cách này được Terraform recommend để cho code clean hơn.
- Với khai báo trên thì nếu không set giá trị cho biến khi chạy `terraform plan`, `terraform apply` thì sẽ sử dụng giá trị detault. Còn nếu muons set giá trị khác thì dùng argument `-var <tên biến>=<giá trị>`:

````sh
terraform plan -var instance_type=t2.large
```

### 3.2. Output values

- Output values cho phép bạn truy cập giá trị của các attributes trong các file Terraform từ automation tools or workflows khác.
- Tạo file `outputs.tf`:

```outputs.tf
output "instance_hostname" {
  description = "Private DNS name of the EC2 instance."
  value       = aws_instance.app_server.private_dns
}
```

>Note: trong Terraform có 2 loại argument: 1. Configurable Arguments (Input Arguments): được set trong các file tf ví dụ: ami, instance_type, tags trong `main.tf`. 2. Computed Attributes (Output / Read-only Attributes) provider tự sinh ra sau khi apply ví dụ private_dns ở trên.

- Sau khi apply có thể xem giá trị các `Output values`:

````sh
terraform output
```

```output
instance_hostname = "ip-172-31-35-26.us-west-2.compute.internal"
```

### 3.3. Modules

- Giống như Module trong code, `Terraform Module` cũng để chia tách, tái sử dụng, đơn giản hóa các infrastructure phức tạp gồm nhiều resources và data.
- Bạn có thể sử dụng các module từ provider
- Ví dụ thêm `module block` trong `main.tf` để cấu hình VPC Network cho EC2:

```main.tf
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.19.0"

  name = "example-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-west-2a", "us-west-2b", "us-west-2c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24"]

  enable_dns_hostnames    = true
}
```

- Cập nhật `resource app_server` trong `main.tf` để sử dụng module

```main.tf
resource "aws_instance" "app_server" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type

  vpc_security_group_ids = [module.vpc.default_security_group_id]
  subnet_id              = module.vpc.private_subnets[0]

  tags = {
    Name = var.instance_name
  }
}
```

- Chạy lại `terraform init` để tải module

````sh
terraform init
```

>Note: terraform sẽ tạo mới workspace khi lần đầu chạy. Các lần sau nếu có provider hay module mới thì `terraform` sẽ tải về.

- Trong thực tế nên tách các module ra file riêng để project dễ mở rộng hơn. Cấu trúc project khuyên dùng:

```txt
project/
├── main.tf              # Gọi các module hoặc resource chính
├── variables.tf         # Khai báo biến (input)
├── outputs.tf           # Xuất giá trị (output)
├── providers.tf         # Cấu hình provider (AWS, Azure,...)
└── modules/
    └── vpc/
        ├── main.tf      # Module logic (VPC, subnet, gateway,...)
        ├── variables.tf
        └── outputs.tf
```

### 3.4. Plan and apply changes

- Build

````sh
terraform fmt
terraform validate
```

- Plan

````sh
terraform plan
```

- Apply

````sh
terraform apply
```

- Kiểm tra state

````sh
terraform state list
```

## 4. Destroy infrastructure

### 4.1. Remove a resource

- Comment block resource `aws_instance.app_server` trong file `main.tf`.
- Ví trong `outputs.tf` có `instance_hostname` phụ thuộc reource `app_server` trong `main.tf` nên cũng comment cả file `outputs.tf`
- Apply

````sh
terraform apply
```

- EC2 sẽ được xóa trên AWS

### 4.2. Destroy workspace

- Xóa toàn bộ infrastructure

````sh
terraform detroy
```
