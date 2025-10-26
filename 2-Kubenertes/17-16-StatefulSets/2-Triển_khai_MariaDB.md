# Triển khai MariaDB trên k8s

- Trên nfs-server (sv5) sửa `/etc/export` thêm `no_root_squash`

```sh
 sudo vi /etc/exports
 # Sửa dòng cuối thành: /data *(rw,sync,no_subtree_check,no_root_squash)
sudo exportfs -rav
sudo systemctl restart nfs-server
```

- Treen Rancher tạo **StatefulSet** `mariadb`

```yml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mariadb
  namespace: ecommerce
spec:
  serviceName: mariadb-service
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      securityContext:
        fsGroup: 65534
      containers:
      - name: mariadb
        image: mariadb:latest
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "devopseduvn"
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mariadb-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mariadb-storage
        persistentVolumeClaim:
          claimName: nfs-pvc
```

>65534 thường tương ứng với nhóm nfsnobody hoặc nobody trong Linux.

- Đã tạo thành công 1 pod **mariadb-0**
![](./images/1.png

- Chỉ cần tăng replica của **SeatefullSet** `mariadb` lên là ta đã có 1 cụm mariadb đảm bảo tính HA

![](./images/2.png)

- Cấu hình Service NodePort MariaDB

```yml
apiVersion: v1
kind: Service
metadata:
  name: mariadb-service
  namespace: ecommerce
spec:
  selector:
    app: mariadb
  type: NodePort
  ports:
    - port: 3306
      targetPort: 3306
      nodePort: 31306
```

- Bây giờ có thể kết nối đến MariaDb ở địa chỉ IP của 2 node sv2 hoặc sv3, port 31306

```sh
mysql -h 192.168.159.103 -P 31306 -u root -p
```
