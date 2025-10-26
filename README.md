# học DevOps từ số 0

## Giới thiệu

Đây là tài liệu tổng hợp về DevOps và Kubernetes, bao gồm:

- Kiến thức cơ bản về Linux
- Quản lý source code với Git/GitLab
- CI/CD với GitLab Runner và Jenkins
- Container hóa ứng dụng với Docker
- Orchestration với Kubernetes
- Các bài thực hành triển khai dự án thực tế

---

## Nội dung

### 1. DevOps For Fresher

#### 1.1. Linux

- [Tạo máy ảo VMware](1-DevOps_For_Fresher/1-Linux/1-Tạo_máy_ảo_VMware.md)
- [Lệnh cơ bản](1-DevOps_For_Fresher/1-Linux/2-Lệnh_cơ_bản.md)
- [Vim](1-DevOps_For_Fresher/1-Linux/3-Vim.md)
- [Thông số](1-DevOps_For_Fresher/1-Linux/4-Thông_số.md)
- [Tài khoản & Quyền](1-DevOps_For_Fresher/1-Linux/5-Tài_khoản_Quyền.md)

#### 1.2. Deploy Project

- [Deploy Project](1-DevOps_For_Fresher/2-Depoly_project/2-Depoly_project.md)

#### 1.3. Git

- [Cài đặt GitLab](1-DevOps_For_Fresher/3-Git/1-Cài_đặt_GitLab.md)

#### 1.4. CI/CD

- [Cài đặt GitLab Runner](1-DevOps_For_Fresher/4-CICD/1-Gitlab_Runner/1.%20Cài_đặt.md)
- [Triển khai Deamon](1-DevOps_For_Fresher/4-CICD/1-Gitlab_Runner/2-Triền_khai_Deamon.md)
- [Triển khai Docker](1-DevOps_For_Fresher/4-CICD/1-Gitlab_Runner/3-Triển_khai_Docker.md)

#### 1.5. Docker

- [Cài đặt Docker](1-DevOps_For_Fresher/5-Docker/1-Cài%20đặt%20docker.md)
- [Dockerfile](1-DevOps_For_Fresher/5-Docker/2-Dockerfile.md)
- [Registry](1-DevOps_For_Fresher/5-Docker/3-Registry.md)

#### 1.6. Jenkins

- [Cài đặt Jenkins](1-DevOps_For_Fresher/6-Jenkins/1-Cài%20đặt.md)
- [CI/CD với Jenkins](1-DevOps_For_Fresher/6-Jenkins/2-CICD.md)
- [CI/CD nâng cao](1-DevOps_For_Fresher/6-Jenkins/3-CICD%20nâng%20cao.md)

---

### 2. Kubernetes

#### 2.1. Kiến trúc Kubernetes

- [Kiến trúc Kubernetes](2-Kubenertes/1-Kiến_trúc_k8s/1-Kiến_trúc_k8s.md)

#### 2.2. Cài đặt Kubernetes

- [Cài đặt Kubernetes On-prem](2-Kubenertes/2-Cài_đặt_k8s/1-On-prem.md)

#### 2.3. Cấu trúc file Kubernetes YAML

- [Cấu trúc file Kubernetes YAML](2-Kubenertes/1-Cấu_trúc_file_k8s_yml/3-Cấu_trúc_file_k8s_yml.md)

#### 2.4. Namespace

- [Namespace](2-Kubenertes/4-Namespace/1-Namespace.md)

#### 2.5. Rancher

- [Cài đặt Rancher](2-Kubenertes/5-Rancher/1-Cài_đặt_rancher.md)
- [Rancher cơ bản](2-Kubenertes/5-Rancher/2-Rancher_cơ_bản.md)

#### 2.6. Pods

- [Pods](2-Kubenertes/6-Pods/1-Pods.md)

#### 2.7. Deployment

- [Deployment](2-Kubenertes/7-Deployment/1-Deployment.md)

#### 2.8. Service

- [Service](2-Kubenertes/8-Service/1-Service.md)

#### 2.9. Ingress

- [Ingress](2-Kubenertes/9-ingress/1-ingress.md)

#### 2.10. Triển khai project Fullstack

- [Triển khai project Fullstack](2-Kubenertes/10-Triển_khai_project_Fullstack/1-Triển_khai_project_Fullstack.md)

#### 2.11. ConfigMap

- [ConfigMap](2-Kubenertes/11-ConfigMap/1-ConfigMap.md)

#### 2.12. Secret

- [Secret](2-Kubenertes/12-Secret/1-Secret.md)

#### 2.13. Request & Limit

- [Request & Limit](2-Kubenertes/13-Request_Limit/1-Request_Limit.md)

#### 2.14. HPA - Horizontal Pod Autoscaler

- [HPA - Horizontal Pod Autoscaler](2-Kubenertes/14-HPA_HorizontalPodAutoscaler/1-HPA_HorizontalPodAutoscaler.md)

#### 2.15. RBAC - Bảo mật

- [RBAC - Role Based Access Control](2-Kubenertes/15-RBAC/1-RBAC.md)

#### 2.16. Storage

- [Storage Class](2-Kubenertes/16-Storage/1-Storage_Class.md)
- [PV & PVC - Persistent Volume & Persistent Volume Claim](2-Kubenertes/16-Storage/2-PV_&_PVC.md)

#### 2.17 StatefullSets

- [StatefullSets](./2-Kubenertes/17-StatefulSets/1-StatefulSets.md)
- [Triển khai MariaDB trên k8s](./2-Kubenertes/17-StatefulSets/2-Triển_khai_MariaDB.md)
- [Triển khai Redis trên k8s](./2-Kubenertes/17-StatefulSets/3-Triển_khai_Reddis.md)

#### 2.18 DaemonSet

- [DảmonSet](./2-Kubenertes/18-DaemonSet/1-DaemonSet.md)
