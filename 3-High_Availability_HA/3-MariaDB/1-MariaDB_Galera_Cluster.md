# MariaDB Galera Cluster

## 1. Caì đặt MariaDB trên 3 sv

- Cài đặt các công cụ cần thiết

```sh
sudo apt update -y
sudo apt install -y net-tools telnet traceroute
 ```

- Cài đặt mariadb server và mariadb client

```sh
sudo apt install -y mariadb-server mariadb-client
```

## 2. Cấu hình mariadb galera cluster

### 2.1. sv1

- Thêm file cấu hình galera

```sh
vi /etc/mysql/conf.d/galera.cnf
```

- Nội dung như sau

```conf
[mysqld]
# Ghi log theo ROW thay vì câu lệnh statement
binlog_format=ROW
# innodb hỗ trợ transaction và lock row
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration

wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Cluster Configuration

wsrep_cluster_name="mariadb-cluster-devopseduvn"
wsrep_cluster_address="gcomm://192.168.159.101,192.168.159.102,192.168.159.103"

# Galera Synchronization Configuration

wsrep_sst_method=rsync

# Node Configuration

wsrep_node_address="192.168.159.101"
wsrep_node_name="sv1"
```

### 2.2. sv2

- Thêm file cấu hình galera

```sh
vi /etc/mysql/conf.d/galera.cnf
```

- Nội dung như sau

```conf
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration

wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Cluster Configuration

wsrep_cluster_name="mariadb-cluster-devopseduvn"
wsrep_cluster_address="gcomm://192.168.159.101,192.168.159.102,192.168.159.103"

# Galera Synchronization Configuration

wsrep_sst_method=rsync

# Node Configuration

wsrep_node_address="192.168.159.102"
wsrep_node_name="sv2"
```

### 2.3. sv3

- Thêm file cấu hình galera

```sh
vi /etc/mysql/conf.d/galera.cnf
```

- Nội dung như sau

```conf
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration

wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Cluster Configuration

wsrep_cluster_name="mariadb-cluster-devopseduvn"
wsrep_cluster_address="gcomm://192.168.159.101,192.168.159.102,192.168.159.103"

# Galera Synchronization Configuration

wsrep_sst_method=rsync

# Node Configuration

wsrep_node_address="192.168.159.103"
wsrep_node_name="sv3"
```

## 3. Cấu hình mariadb galera cluster

- Tắt mariadb server **sv1**

```sh
sudo systemctl stop mariadb
```

- Khởi tạo cụm mariadb galera cluster  **sv1**

```sh
 galera_new_cluster
 ```

- Kiểm tra cụm mariadb galera cluster (thấy số lượng 1)  **sv1**

```sh
mysql -u root -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```

```sh
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 1     |
+--------------------+-------+
```

- Khởi động lại mariadb server để áp dụng cấu hình mới join vào cụm trên **sv2 và sv3**

```sh
sudo systemctl restart mariadb
```

- Kiểm tra lại trên **sv1**

```sh
root@sv1:~# mysql -u root -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
```

## 4. Kiểm tra tính đồng bộ dữ liệu

- Trên **sv1**

```sh
mysql
```

- Thêm dữ liệu

```sql
create database devopseduvndb;
use devopseduvndb;
create table series (Name VARCHAR(50) PRIMARY KEY, Lever VARCHAR(25), Release_date DATE);
insert into series (Name, Lever, Release_date) VALUES ('DevOps for Freshers', 'Basic/Freshers', '2024-01-15'), ('Xây dựng quy trình pipeline DevSecOps', 'Thực tế chuyên sâu', '2024-06-16'), ('HA tools', 'Advanced/Thực tế', '2024-08-18');
```

- Thực hiện trên sv2 hoặc sv3

```sh
#mysql
```

- Kiểm tra dữ liệu

```sql
use devopseduvndb;
show databases;
show tables;
select * from series;
```

## 4. Cài đặt Maxscale để quản lý cụm mariadb hoặc mysql

- Cài đặt **maxscale** từ nguồn chính thức (Thực hiện trên cả 3 sv)

```sh
wget https://dlm.mariadb.com/2344079/MaxScale/6.4.1/packages/ubuntu/jammy/x86_64/maxscale-6.4.1-1.ubuntu.jammy.x86_64.deb

dpkg -i maxscale-6.4.1-1.ubuntu.jammy.x86_64.deb
```

- Mở kết nối mariadb đến anywwhere trên cả 3 sv

```sh
vi /etc/mysql/mariadb.conf.d/50-server.cnf
```

>Sửa bind-address = 0.0.0.0

```sh
=
```

- Tạo user maxscale trong mariadb cho maxscale (**sv1**)

```sh
mysql
```

```sql
-- @'%' là giới hạn IP mà user có thể truy cập
CREATE USER 'maxscale'@'%' IDENTIFIED BY 'devopseduvn';
-- Gán truyền truy cập trạng thái các raplication
GRANT REPLICATION CLIENT ON *.* TO 'maxscale'@'%';
GRANT REPLICATION SLAVE ADMIN ON *.* TO 'maxscale'@'%';
GRANT REPLICA MONITOR ON *.* TO 'maxscale'@'%';
FLUSH PRIVILEGES;
```

- Cấu hình lại mariadb và maxscale

```sh
# Backup file cnf để dự phòng
cp /etc/maxscale.cnf /etc/maxscale.cnf.org
# Sửa file /etc/maxscale.cnf
vi /etc/maxscale.cnf
```

- Chinhr swar
  - Copy ra 3 block [Serrver] vaf swar address với ip tương ứng.
  - [MariaDB-Monitor]
    - servers=server1,server2,server3
    - user=maxscale
    - password=devopseduvn
  - [Read-Only-Service]
    - password=devopseduvn
    - [Read-Only-Service]
  - [Read-Write-Service]
    - servers=server1,server2,server3
    - user=maxscale
    - password=devopseduvn

- Start maxscale

```sh
sudo systemctl start maxscale
maxctrl list servers
```

- Ssửa xong ở sv1 chạy thử maxsacle ok thì copy sang sv2 và sv3
=

```sh
scp /etc/maxscale.cnf root@sv2:/etc/maxscale.cnf
scp /etc/maxscale.cnf root@sv3:/etc/maxscale.cnf
```

- Khởi động maxscale trên **sv2 và sv3**

```sh
```sh
sudo systemctl start maxscale
maxctrl list servers
```

>Note: Thử reboot sv1 xem