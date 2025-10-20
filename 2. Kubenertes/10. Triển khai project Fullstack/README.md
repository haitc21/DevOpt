# Triển khai dự án Fullstack trên k8s

## 1. Plan

- Dự án Fullstack bao gồm:
  - BE: Java Spring Boot - [api-ecommerce.devopsedu.vn](http://api-ecommerce.devopsedu.vn)
  - FE: Angular - [ecommerce.devopsedu.vn](http://ecommerce.devopsedu.vn)
  - DB: MariaDB - Cài standalone trên **sv5** cho tiết kiệm tài nguyên
- Source code trong ./Fullstack-Ecommerce-Web
  - 01-starter-files_db-scripts: Để init db
  - 02-backend_spring-boot-rest-api: BE
  - 03-frontend_angular-ecommerce: FE

## 2. Cài đặt DB

- Trên **sv5**

```sh
# Install
sudo apt update -y
sudo apt install mariadb-server -y
#  Mở kết nối
# Sửa bind-addres = 127.0.0.1s => 0.0.0.0
vi /etc/mysql/mariadb.conf.d/50-server.cnf
# restart mariadb
sudo systemctl restart mariadb
# Truy cập
mysql
show databases;
```

- Chạy tất cả các file trong `./Fullstack-Ecommerce-Web/01-starter-files_db-scripts` để tạo db và thêm dữ liệu.

```sh
show tables;
```

### 3. Triển khai FE

- Build image

```sh
# cd vào thư mục FE
cd 03-frontend_angular-ecommerce
# Build image
docker build -t ecommerce-frontend:v1
# Kiểm tra
docker images
# Login dockerhub
docker login
# đánh lại tag thêm <username>
docker tag ecommerce-frontend:v1 haitc21/ecommerce-frontend:v1
# Pussh leen dockerhub
docker push haitc21/ecommerce-frontend:v1
```

- Kiểm tra trên [https://hub.docker.com/](https://hub.docker.com/)
- Tạo project và namespace trên rancher

![](./images/1.png)

- Import lên rancher file yml sau chính là file yml mẫu ở phần trước [Ingress](../9.%20ingress/README.md) sửa:
  - Đổi name `car-serv`=> `ecommerce-frontend`.
  - namespace: ecommerce
  - image: hiatc21/ecommerce-frontend:v1
  - host: ecommerce.devopsedu.vn

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ecommerce-frontend
  name: ecommerce-frontend-deployment
  namespace: ecommerce
spec:
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: ecommerce-frontend
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecommerce-frontend
      namespace: ecommerce
    spec:
      containers:
        - image: docker.io/haitc21/ecommerce-frontend:v1
          imagePullPolicy: Always
          name: ecommerce-frontend
          ports:
            - containerPort: 80
              name: tcp
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: ecommerce-frontend-service
  namespace: ecommerce
spec:
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - name: tcp
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: ecommerce-frontend
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-frontend-ingress
  namespace: ecommerce
spec:
  ingressClassName: nginx
  rules:
    - host: ecommerce.devopsedu.vn
      http:
        paths:
          - backend:
              service:
                name: ecommerce-frontend-service
                port:
                  number: 80
            path: /
            pathType: Prefix
```

- Add host: 192.168.159.105 ecommerce.devopsedu.vn
- Kiểm tra [ecommerce.devopsedu.vn](http://ecommerce.devopsedu.vn/)

## 4. Triển khai BE

- Sửa file [application.properties](./Fullstack-Ecommerce-Web/02-backend_spring-boot-rest-api/src/main/resources/application.properties) trỏ đúng IP của mariadb cài ở phần 2 là 192.168.159.105:3306
- Build image `haitc21/ecommerce-backend:v1`
```sh
cd 02-backend_spring-boot-rest-api
ocker build -t ecommerce-backend:v1 .
ocker tag ecommerce-backend:v1 haitc21/ecommerce-backend:v1
docker push haitc21/ecommerce-backend:v1
```
- Deploy:
  - Thay `frontend` thành `backend`
  - Đổi host thành `api-ecommerce.devopsedu.vn`
  - Expose port 8080

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ecommerce-backend
  name: ecommerce-backend-deployment
  namespace: ecommerce
spec:
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: ecommerce-backend
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecommerce-backend
      namespace: ecommerce
    spec:
      containers:
        - image: docker.io/haitc21/ecommerce-backend:v1
          imagePullPolicy: Always
          name: ecommerce-backend
          ports:
            - containerPort: 8080
              name: tcp
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: ecommerce-backend-service
  namespace: ecommerce
spec:
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - name: tcp
      port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    app: ecommerce-backend
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-backend-ingress
  namespace: ecommerce
spec:
  ingressClassName: nginx
  rules:
    - host: api-ecommerce.devopsedu.vn
      http:
        paths:
          - backend:
              service:
                name: ecommerce-backend-service
                port:
                  number: 8080
            path: /
            pathType: Prefix
```

- Add host: 192.168.159.105 api-ecommerce.devopsedu.vn
- Test: http://api-ecommerce.devopsedu.vn/api/products