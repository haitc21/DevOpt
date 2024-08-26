1. Cài đặt gitlab runner

- [Install Git Lab runner on Ubuntu](https://www.linuxtechi.com/how-to-install-gitlab-runner-on-ubuntu/)

``` sh
sudo -i
apt update
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
apt install gitlab-runner
gitlab-runner -version
```

- Kiểm tra user gitlab-runner

``` sh
vi /etc/passwd
```

![](./image/1.png)

2. Cấu hình CI/CD trên gitlab

- tạo group
- Vào trang chủ gitlab => đăng nhập => projects => create new project => create blank project
- tạo 2 nhánh develop và release. Release chỉ maintainer mới được push
- Thêm 2 user dev1 có reole developer và lead1 role maintainer
- Vào dự án => Settings => CICD => Runner rồi ấn Extend
- Lên server đã cài gitlab-runner

``` sh
gitlab-runner register
```

- Url là domain của gitlab ở đây là: <http://gitlab.haitc.local/>
- Token
![](./image/2.png)

- Description và tag để tên server
- excutor để shell
![](./image/3.png)

- Mở file config

``` sh
vi /etc/gitlab-runner/config.toml
```

- Sửa concurrent = 4 (số dự án mà con runner chạy được)
![](./image/4.png)

- Khởi động runner

``` sg
nohub gitlab-runner run --working-directory /home/gitlab-runner --config /etc/gitlab-runner/config.toml --service gitlab-runner --user gitlab-runner 2>&1 &
```

- Kiểm tra process

``` sh
gitlab-runner 2>&1
```

![](./image/5.png)

- Quay lại trang gitlab và đã thấy 1 runner mới tên là gitlab-server
![](./image/6.png)
- Edit
![](./image/7.png)

3. Viết kịch bản

- tạo file .gitlab-ci.yml trong thư mục gốc của dự án

``` yml
stages:
  - build
  - deploy
  - checklog

build:
  stage: build
  script:
    - whoami
    - pwd
    - ls
  tags:
    - gitlab-server

```

- Build => Pipeline
![](./image/8.png)

- Chi tiết job
![](./image/9.png)

>NOTE: Khi chỉnh sửa bất cứ file nào thì pipeline sẽ được chạy. Sẽ xóa toàn bộ code cũ đi và downloa code mới.Qua mỗi stage cũng xóa đi down lại

- Cho usser gitlab-runner chạy với quyền sudo với các câu lẹnh cp, chown, su java1 mà không cần nhập password

``` sh
sudo víudo
```

- Thêm vào phần # User privilege specification

```
gitlab-runner ALL=(ALL) NOPASSWD: /bin/cp*
gitlab-runner ALL=(ALL) NOPASSWD: /bin/chown*
gitlab-runner ALL=(ALL) NOPASSWD: /bin/su java1*
```

-Thêm stage deploy

``` yml
variables:
  projectuser: java1
  projectname: demo
  projectversion: 0.0.1-SNAPSHOT
  projectpath: /projects/$projectuser/

stages:
  - build
  - deploy
  - checklog

build:
  stage: build
  variables:
    GIT_STRATEGY: clone
  script:
    - cd src/demo/
    - mvn clean install -DskipTests=true
  tags:
    - gitlab-server

deploy:
  stage: deploy
  variables:
    GIT_STRATEGY: none
  script:
    - sudo cp src/demo/target/$projectname-$projectversion.jar $projectpath
    - sudo chown -R $projectuser. $projectpath
    - sudo su $projectuser -c "kill -9 $(ps -ef | grep $projectname-$projectversion.jar | grep -v grep | awk '{print $2}')"
    - sudo su $projectuser -c "cd $projectpath; nohup java -jar $projectname-$projectversion.jar > nohup.out 2>&1 &"
  tags:
    - gitlab-server

```

>NOTE: câu lệnh kill chỉ chạy từ lần 2 trở đi vì khi đó có process cũ đang chạy cần kill đi

- thêm only tags để chỉ chạy khi có tag mới

``` yml
variables:
  projectuser: java1
  projectname: demo
  projectversion: 0.0.1-SNAPSHOT
  projectpath: /projects/$projectuser/

stages:
  - build
  - deploy
  - checklog

build:
  stage: build
  variables:
    GIT_STRATEGY: clone
  script:
    - cd src/demo/
    - mvn clean install -DskipTests=true
  tags:
    - gitlab-server
  only:
  - tags

deploy:
  stage: deploy
  variables:
    GIT_STRATEGY: none
  script:
    - sudo cp src/demo/target/$projectname-$projectversion.jar $projectpath
    - sudo chown -R $projectuser. $projectpath
    - sudo su $projectuser -c "kill -9 $(ps -ef | grep $projectname-$projectversion.jar | grep -v grep | awk '{print $2}')"
    - sudo su $projectuser -c "cd $projectpath; nohup java -jar $projectname-$projectversion.jar > nohup.out 2>&1 &"
  tags:
    - gitlab-server
  only:
  - tags
```

- Thêm stage sholog

```yml
# Khai báo biến global
variables:
  projectuser: java1
  projectname: demo
  projectversion: 0.0.1-SNAPSHOT
  projectpath: /projects/$projectuser/

# Khai báo tên các bước
stages:
  - build
  - deploy
  - checklog

# Bước 1 build dữ ạ
build:
#tên stage trùng với ở trên
  stage: build
  #biến nội bộ trong stage build
  variables:
  #Xóa code cũ clone code mới
    GIT_STRATEGY: clone
  script:
    - cd src/demo/
    #build dự án java
    - mvn clean install -DskipTests=true
  tags:
  #Chỉ định runner có tag là gitlab-runner
    - gitlab-server
  only:
  #Chỉ khi repo có tag mới thì mới run ci. Ví dụ tag v1.0
  - tags
#Deploy dự án
deploy:
  stage: deploy
  variables:
  #Không cần clone code
    GIT_STRATEGY: none
  script:
  #copy file jar sang thư mục deploy
    - sudo cp src/demo/target/$projectname-$projectversion.jar $projectpath
    #Chỉnh sửa quyền
    - sudo chown -R $projectuser. $projectpath
    #Khi deployu lần 2 trở đi thì cần kill process cũ
    #grep -v grep: Vì khi search sẽ ra 2 kết quả mà kết quả thứ 2 chính là process tìm kiếm nên cần bỏ kết quả đó
    #awk '{print $2}')": lấy kết quả ở cột 2 chính là PID
    - sudo su $projectuser -c "kill -9 $(ps -ef | grep $projectname-$projectversion.jar | grep -v grep | awk '{print $2}')"
    #Chạy dự án lưu log vào file nohp.out
    - sudo su $projectuser -c "cd $projectpath; nohup java -jar $projectname-$projectversion.jar > nohup.out 2>&1 &"
  tags:
    - gitlab-server
  only:
  - tags

checklog:
  stage: checklog
  variables:
    GIT_STRATEGY: none
  script:
  #Xem log trong file nohup.out
    - sudo su $projectuser -c "cd $projectpath; tail -n 10000 nohup.out"
  tags:
    - gitlab-server
  only:
  - tags

```

- cấu hình thêm [Continuous Deliver](https://www.youtube.com/watch?v=y-ByTDU_Lg8&list=PLsvroIvFNP1KU8foUeCC-hbJbqnAggWL2&index=18)
  - when: manual: Chạy stage thủ công
  - if [ "$GITLAB_USER_LOGIN" == 'lead1' ]; Chỉ user lead1 mới có thể chạy stage deploy

``` yml
# Khai báo biến global
variables:
  projectuser: java1
  projectname: demo
  projectversion: 0.0.1-SNAPSHOT
  projectpath: /projects/$projectuser/

# Khai báo tên các bước
stages:
  - build
  - deploy
  - checklog

# Bước 1 build dữ ạ
build:
  #tên stage trùng với ở trên
  stage: build
  #biến nội bộ trong stage build
  variables:
    #Xóa code cũ clone code mới
    GIT_STRATEGY: clone
  script:
    - cd src/demo/
    #build dự án java
    - mvn clean install -DskipTests=true
  tags:
    #Chỉ định runner có tag là gitlab-runner
    - gitlab-server
  only:
    #Chỉ khi repo có tag mới thì mới run ci. Ví dụ tag v1.0
    - tags
#Deploy dự án
deploy:
  stage: deploy
  variables:
    #Không cần clone code
    GIT_STRATEGY: none
  when: manual
  script:
    >
      if [ "$GITLAB_USER_LOGIN" == 'lead1' ]; then
        sudo cp src/demo/target/$projectname-$projectversion.jar $projectpath
        sudo chown -R $projectuser. $projectpath
        sudo su $projectuser -c "kill -9 $(ps -ef | grep $projectname-$projectversion.jar | grep -v grep | awk '{print $2}')"
        sudo su $projectuser -c "cd $projectpath; nohup java -jar $projectname-$projectversion.jar > nohup.out 2>&1 &"
      else
        echo "Permission denined!"
        exit 1
      fi
  tags:
    - gitlab-server
  only:
    - tags

checklog:
  stage: checklog
  variables:
    GIT_STRATEGY: none
  when: manual
  script:
    #Xem log trong file nohup.out
    - sudo su $projectuser -c "cd $projectpath; tail -n 10000 nohup.out"
  tags:
    - gitlab-server
  only:
    - tags

```
