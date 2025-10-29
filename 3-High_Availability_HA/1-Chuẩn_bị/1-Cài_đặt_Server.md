# Giới thiệu

Seri hướng dẫn triển khai các công cụ quen thuộc đảm bảo tính HA.

Chuẩn bị:

| Server Name | IP Address        | Disk Size | Processors | Cores per Processor | Total Cores | Memory (RAM) | Network Type |
|--------------|------------------|------------|-------------|----------------------|--------------|----------------|----------------|
| sv1          | 192.168.159.101  | 20 GB      | 2           | 1                    | 2            | 4 GB           | NAT            |
| sv2          | 192.168.159.102  | 20 GB      | 2           | 1                    | 2            | 4 GB           | NAT            |
| sv3          | 192.168.159.103  | 20 GB      | 2           | 1                    | 2            | 4 GB           | NAT            |

---