# 1. Giới thiệu

- Terraform là công cụ mã nguồn mở do HashiCorp phát triển.
- Terraform tạo và quản lý các resource trên cloud dựa trên các API của n provider đó.

![](https://web-unified-docs-hashicorp.vercel.app/api/assets/terraform/latest/img/docs/intro-terraform-apis.png)

- Các provider phổ biến: Amazon Web Services (AWS), Azure, Google Cloud Platform (GCP), Kubernetes, Helm, GitHub, Splunk, DataDog. Có thể xem toàn bộ public provider trên [Terraform Registry](https://registry.terraform.io/?product_intent=terraform)

- Infrastructure as Code (IaC): Sử dụng code (tf file) để triển khai, cấu hình cơ sở hạ tầng.Giúp triển khai, cấu hình cơ sử hạ tầng chính xác (tay vì thao tác trên Web UI như AWS Web Console thì sử dụng code sẽ tránh sai sót khi thao tác), tính tái sử dụng, triển khai tự động thông qua các tool như Ansible.

## 1.1 Terraform Workflow

![](https://web-unified-docs-hashicorp.vercel.app/api/assets/terraform/latest/img/docs/intro-terraform-workflow.png)
 Gồm 3 bước:

- **Write**: Người dùng viết các file tf định nghĩa các resource (máy ảo EC2, kho lưu trữ AWS)
- **Plan**: Teraform đưa ra `execution plan` thể hiện các infrastructure được creae, update, deploy trên các provider được cáu hihf trong bước **Write**.
- **Apply**: Teraform thực thi **Plan** và trả về **state file**

### 1.3 Tại sao dùng Terrafrom

- Có thể quản lý rất nhiều loại infrastructure
- Theo dõi các infrastructure: Quản lý state qua state file
- Tự động tay đỏi infrastucture theo file cấu hình (main.tf)
- Tiêu chuẩn hóa cấu hình, có tính tái sử dụng qua module
- Dễ dàng quản lý phiên bản của cấu hình (tf file) bằng SCM, sử dụng ![HCP Terraform](https://developer.hashicorp.com/terraform/intro/terraform-editions#hcp-terraform) để chạy, quản lý team, phiên bản, private registry...

- Cộng đồng lớn

## 2. Cài đặt
>
>Node: Hướng dẫn này dành cho Window các OS khác làm hướng dẫn của tài liệu chính thức ![Install Teraform](https://developer.hashicorp.com/terraform/install)
>
### 2.2 Cài đặt Terraform

- ![Tải Teraform 1.13.3](https://releases.hashicorp.com/terraform/1.13.3/terraform_1.13.3_windows_386.zip)
- Giải nén
- Thêm thư mục vào PATH
- Kiểm tra

```cmd
teraform --version
```

### 2.1 TFLint

- ![TFLint](https://github.com/terraform-linters/tflint#installation)

- Cài đặt ![Chocolatey](https://chocolatey.org/install#individual) (Yều cầu .Net Framework 4.8 thường thì được cài sẵn trong window 11): Chocolatey là 1 package manager trên Window tương tự như apt trong Ubuntu.
- Chạy PowerShell với quyền Admintrator

```ps1
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

- Cài đặt TFLint

```ps1
choco install tflint[]
tflint --version 
```
