# 1. Cấu trúc file yml trong k8s

## 1.1. aqpiVersion

- Khai báo apiVersion tương ứng với các tài nguyên trong k8s
Ví dụ:
- apiVersion: v1 -> root
- apiVersion: apps/v1 -> deployment

## 1.2. kind

- Khai báo các tài nguyên trong k8s
Ví dụ
- kind: Pod
- kind: Deployment
- kind: Service
- kind: Ingress

## 1.3. metadata

- Chứa các thông tin liên quan đến tài nguyên

```yml
metadata:
  name:
  namespace:
  labels:
  annotations:
```

## 1.4. spec

- Định nghĩa các cấu hình cụ thể của tài nguyên

# 2. Ví dụ

## 2.1 Khai báo pod chạy nginx

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.25
      ports:
        - containerPort: 80
```

## 2.2. Deployment (quản lý nhiều Pod)

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
```

## 2.3. Service expose Pod ra ngoài cluster

```yml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
  type: NodePort

```
