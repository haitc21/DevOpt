# 1. Giới thiệu

- Terraform là công cụ mã nguồn mở do HashiCorp phát triển.
- Terraform tạo và quản lý các resource trên cloud dựa trên các API của n provider đó.

![](https://web-unified-docs-hashicorp.vercel.app/api/assets/terraform/latest/img/docs/intro-terraform-apis.png)

- Các provider phổ biến: Amazon Web Services (AWS), Azure, Google Cloud Platform (GCP), Kubernetes, Helm, GitHub, Splunk, DataDog. Có thể xem toàn bộ public provider trên [Terraform Registry](https://registry.terraform.io/?product_intent=terraform)

## 1.1 Terraform Workflow

![](https://web-unified-docs-hashicorp.vercel.app/api/assets/terraform/latest/img/docs/intro-terraform-workflow.png)
 Gồm 3 bước:

- **Write**: Người dùng viết các file tf định nghĩa các resource (máy ảo EC2, kho lưu trữ AWS)
- **Plan**: Teraform đưa ra `execution plan` thể hiện các infrastructure được creae, update, deploy trên các provider được cáu hihf trong bước **Write**.
- **Apply**: Teraform thực thi **Plan** và trả về **state file**

### 1.3 tại sao dùng Terrafrom

- Có thể quản lý rất nhiều loại infrastructure
- Theo dõi các infrastructure: Quản lý state qua state file
- Tự động tay đỏi infrastucture theo file cấu hình (main.tf)
- Tiêu chuẩn hóa cấu hình, có tính tái sử dụng qua module
- Dễ dàng quản lý phiên bản của cấu hình (tf file) bằng SCM, sử dụng ![HCP Terraform](https://developer.hashicorp.com/terraform/intro/terraform-editions#hcp-terraform) để chạy, quản lý team, phiên bản, private registry...

- Cộng đồng lớn
