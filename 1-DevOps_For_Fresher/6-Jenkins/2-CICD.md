# 1. Jenkins Agent

- Jenkins Agent để kết nối jenkins với các server deploy
- Cần cài dặt java cùng phiên bản với jenkins server

``` sh
sudo apt install fontconfig openjdk-17-jre -y
```

- tạo user jenkins không cần mật khẩu

``` sh
sudo adduser --quiet jenkins --disabled-password --gecos "Jenkins user"
```

- tạo thư mục làm việc /var/lib/jenkins

``` sh
sudo mkdir -p /var/lib/jenkins
sudo chown jenkins. /var/lib/jenkins
su jenkins
cd /var/lib/jenkins
```

- Agent cần 1 port trên jenkins server. Kiểm tra trên jenkins server có port trống không
![](./image/2.2.png)
- Trong trang chủ jenkins vào Manage Jenkins => Security => Agents
![](./image/2.3.png)
- truy cập trang chủ jenkins => Manage Jenkins => Node => New Node
![](./image/2.1.png)

>NOTE: Chú ý cấu hình Number of executors và Remote root directory
![](./image/2.4.png)

- Nhấn vào Node vừa được sẽ có hướng dẫn cấu hình. Nên sử dụng câu lệnh ở dưới
![](./image/2.5.png)

- Ví dụ câu lệnh như sau:

``` sh
cho fe59a2ed83c7ccbe5a2be50d6668cef294193f0b20b85bca574aa84c295a6da0 > secret-file
curl -sO http://jenkins.haitc.local/jnlpJars/agent.jar
java -jar agent.jar -url http://jenkins.haitc.local/ -secret @secret-file -name "lab-server" -workDir "/var/lib/jenkins"
```

## 2. Tạo Jenkins Agent Service

### Bước 1: Tạo file script khởi động cho Jenkins Agent

- Tạo một file script tại /usr/local/bin/jenkins-agent.sh với nội dung phù hợp để sử dụng thư mục /var/lib/jenkins làm thư mục làm việc:

``` sh
sudo nano /usr/local/bin/jenkins-agent.sh
```

``` jenkins-agent.sh
#!/bin/bash

# Thư mục làm việc của Jenkins agent
WORKDIR="/var/lib/jenkins"

# Chuyển đến thư mục làm việc
cd $WORKDIR

# Tải file agent.jar nếu chưa có
if [ ! -f "$WORKDIR/agent.jar" ]; then
    curl -sO http://jenkins.haitc.local/jnlpJars/agent.jar -o $WORKDIR/agent.jar
fi

# Chạy Jenkins agent
java -jar $WORKDIR/agent.jar -url http://jenkins.haitc.local/ -secret @secret-file -name "lab-server" -workDir "$WORKDIR"
```

### Bước 2: Đặt quyền thực thi cho file script

``` sh
sudo chmod +x /usr/local/bin/jenkins-agent.sh
```

### Bước 3: Tạo file service cho Jenkins Agent

- Tạo một file service tại /etc/systemd/system/jenkins-agent.service:

``` sh
sudo nano /etc/systemd/system/jenkins-agent.service
```

``` jenkins-agent.service
[Unit]
Description=Jenkins Agent
After=network.target

[Service]
Type=simple
User=jenkins
Environment="JENKINS_URL=http://jenkins.haitc.local/"
Environment="JENKINS_SECRET_FILE=/var/lib/jenkins/secret-file"
WorkingDirectory=/var/lib/jenkins
ExecStart=/usr/local/bin/jenkins-agent.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

- After=network.target: Đợi network khởi động xong mới chạy service.

### Bước 4: Đặt secret-file trong thư mục làm việc

- Tạo và lưu secret-file với nội dung vào thư mục /var/lib/jenkins:

``` sh
echo fe59a2ed83c7ccbe5a2be50d6668cef294193f0b20b85bca574aa84c295a6da0 | sudo tee /var/lib/jenkins/secret-file
```

### Bước 5: Tải lại systemd và kích hoạt dịch vụ

- Chạy các lệnh sau để tải lại daemon systemd, kích hoạt và khởi động dịch vụ Jenkins agent:

``` sh
sudo systemctl daemon-reload
sudo systemctl enable jenkins-agent
sudo systemctl start jenkins-agent
```

### Bước 6: Kiểm tra trạng thái của dịch vụ

- Kiểm tra trạng thái của dịch vụ để đảm bảo nó đang chạy:

``` sh
sudo systemctl status jenkins-agent
```

# 2.Kết nói jenkins => gitlab

- Cài đặt 2 plugin là gitlab và BlueOcean: Manage Jenkins => Plugin => Availible Plugin

![](./image//2.7.png)

- Ấn Install => Kéo xuống cuối tick vào Restart
- tạo tài khoản jenkins có quyền admintrator trong trang gitlab
- Đăng nhập gitlab bằng tài khoản jenkins => Edit Profile => Access Tokens => Add new token
![](./image/2.8.png)

- Manage Jenkins => System => Gitlab: Điền thông tin gitlab
- Phần Credential ấn vào Add => Jenkins =>
![](./image//2.9.png)
- test connection => Save

# 3. tạo pipeline

- Trang chủ jenkins => New item
![](./image/2.6.png)
- tạo folder
- Vào Folder tạo ở trên là demojava => New Item => Pipeline
- tick **Discard old builds ?**
- **Max # of builds to keep** : 10
- Chọn **GitLab Connection** đã cấu hình trong System
- trong **Build Triger** tick **Build when a change is pushed to GitLab. GitLab webhook URL: <http://jenkins.haitc.local/project/java1>**
![](./image/2.10.png)
- Phần **Pipelin** => Definition: Pipeline script from SCM
- SCM: Git
- Repository URL: <http://gitlab.haitc.local/cidc-basic/java1>
- Tạo Credential để xác thực
![](./image/2.11.png)
- **Branch Specifier (blank for 'any')** cấu hình nhánh trong repo
- **Script Path**: Jenkinsfile
- Save

# 4. Cấu hình Webhock trong project GitLab

- Vào trang gitlab => Admin => Settings => network => Outbound requests => tick **Allow requests to the local network from webhooks and integrations**

- Vào project gitlab => Settings => Webhocks => Add new webhock
- Cấu trúc URL:

```
http://<user trong jenkins>:<token của user đó>@<địa chỉ jenkins>/project/<đường dẫn dự án trên jenkins>
```

- Vào trang jenkins ấn vào tên tài khoản góc trên bên trái => Configure => Add token => Generate
- Project name lấy bằng cách vào Folder => Ấn vào Pipelin
![](./image/2.12.png)
- Giá trị URL

```
http://admin:11224fda86f50fa28cb2b68b8a3560f11a@jenkins.haitc.local/project/cidc-basic/java1
```

- Chọn theo hình nhớ **bỏ tick SSL** vì mỗi trường lab dùng HTTP
![](./image/2.13.png)
![](./image/2.14.png)
![](./image/2.15.png)
![](./image/2.16.png)

# 5. Jenkinsfile

- Jenkinsfile sử dụng ngôn ngữ **Groovy** nên nó có đầy đủ syntax mạnh mẹ hơn so với file yml

``` Jenkinsfile
pipeline {
    agent {
        label 'lab-server'
    }
    stages {
        stage('info') {
            steps {
                script {
                    sh '''
                        whoami
                        pwd
                        ls -la
                    '''
                }
            }
        }
    }
}
```

# 6. CICD docker

- Cài đặt Docker trên Jenkins  Agent (lab-server)
- thêm user jenkins vào group docker

``` sh
sudo usermod -aG docker jenkins
```

- Thêm enviroment variable trên jenkins: Trang chủ Jenkins => Manage Jenkins => System => Global properties =>  Environment variables => Tạo các biến REGISTRY_PASSWD, REGISTRY_PROJECT, REGISTRY_URL, REGISTRY_USER

## 6.1 Tự động deploy: Continuous Deployment

- Cách này chỉ nên sử dụng trong môi trường dev, stagging, test vì sẽ khó kiểm soát code đẩy lên
- Jenkinsfile có thể bỏ stage('Info')

``` Jenkinsfile 
pipeline {
    agent { label 'lab-server' }
    environment {
        DOCKER_IMAGE = "${env.REGISTRY_URL}/${env.REGISTRY_PROJECT}/${env.JOB_NAME}:${env.GIT_COMMIT.substring(0, 7)}"
        DOCKER_CONTAINER = 'demojavaapp'
    }
    stages {
        stage('Info') {
            steps {
                script {
                    sh '''
                        whoami
                        echo "Hostname: $(hostname)"
                        echo "REGISTRY_PROJECT: $REGISTRY_PROJECT"
                        echo "REGISTRY_URL: $REGISTRY_URL"
                        echo "DOCKER_IMAGE: $DOCKER_IMAGE"
                        echo "DOCKER_CONTAINER: $DOCKER_CONTAINER"
                    '''
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    sh 'echo "$REGISTRY_PASSWD" | docker login $REGISTRY_URL -u $REGISTRY_USER --password-stdin'
                    sh 'docker build -t $DOCKER_IMAGE ./src/demo'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sh 'docker pull $DOCKER_IMAGE'
                    sh 'docker rm -f $DOCKER_CONTAINER || true'
                    sh 'docker run --name $DOCKER_CONTAINER -d -p 8083:8081 $DOCKER_IMAGE'
                }
            }
        }
        stage('Check Logs') {
            steps {
                script {
                    sh 'docker logs $DOCKER_CONTAINER'
                }
            }
        }
    }
}

```

## 6.2 Sử dụng Credential để bảo mật thông tin

- Để bảo mật hơn cho REGISTRY_USER và REGISTRY_PASSWD nên sử dụng Credential thay vì sử dụng Environment variables
- tạo Credential có id là docker_rdocker_registry_credsegistry: Trang chủ jenkins => manage Jenkins => Credentials => Click vào chữ System

![](./image/2.18.png)

- Chọn domain

![](./image/2.19.png)

- Add Credential có id là docker_registry_creds

![](./image/2.20.png)

![](./image/2.21.png)

- Chỉnh sẳ lại stage Build

``` Jenkinsfile
        stage('Build') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker_registry_creds', usernameVariable: 'REGISTRY_USER', passwordVariable: 'REGISTRY_PASSWD')]) {
                        sh 'echo "$REGISTRY_PASSWD" | docker login $REGISTRY_URL -u $REGISTRY_USER --password-stdin'
                        sh 'docker build -t $DOCKER_IMAGE ./src/demo'
                        sh 'docker push $DOCKER_IMAGE'
                    }
                }
            }
        }
```

- Xóa 2 Enviroment variable: REGISTRY_USER và REGISTRY_PASSWD
- Chạy lại pipeline

![](./image/2.22.png)

## 6.3 Xác nhận khi deploy: Continuous Delivery

- Nên sử dụng trong Production vì khi delpy cần người xác nhận đảm bảo tính đúng đắn
- Ví dụ dưới đây cho phép chờ 30 phút đợi admin vào xác nhận mới deploy
- Chỉnh sửa lại stage deploy

``` Jenkinsfile
        stage('Deploy') {
            steps {
                script {
                    try {
                        timeout(time: 30, unit: 'MINUTES') {
                            env.useChoice = input(
                                message: 'Can it be deployed?',
                                parameters: [choice(
                                    name: 'deploy',
                                    choices: 'no\nyes',
                                    description: 'Choose "yes" if you want to deploy')]
                            )
                        }
                        if (env.useChoice == 'yes') {
                            sh 'docker pull $DOCKER_IMAGE'
                            sh 'docker rm -f $DOCKER_CONTAINER || true'
                            sh 'docker run --name $DOCKER_CONTAINER -d -p 8083:8081 $DOCKER_IMAGE'
                        } else {
                            echo 'Deployment not confirmed. Stopping progress!'
                        }
                    } catch (Exception ex) {
                        echo "An error occurred: ${ex.getMessage()}"
                    }
                }
            }
        }
```

![](./image/2.17.png)

## 6.4 Nâng cập Jenkinsfile trở nên Pro hơn

- Thêm GIT_TAG
- Thêm label cho câu lệnh sh
- tách các script thành các Method
- Sau khi deply thành công xóa các images cũ

``` Jenkinsfile
pipeline {
    agent { label 'lab-server' }
    environment {
        DOCKER_CONTAINER = 'demojavaapp'
        DOCKER_IMAGE_NAME = "${env.REGISTRY_URL}/${env.REGISTRY_PROJECT}/${env.JOB_NAME}"
        GIT_TAG = sh(returnStdout: true, script: 'git tag --sort version:refname | tail -1').trim()
        DOCKER_TAG = "${GIT_TAG ?: 'notag'}_${env.GIT_COMMIT.substring(0, 7)}"
        DOCKER_IMAGE = "${DOCKER_IMAGE_NAME}:${DOCKER_TAG}"
    }
    stages {
        stage('Info') {
            steps {
                script {
                    printInfo()
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    dockerLogin()
                    buildAndPushDockerImage()
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    deployDockerContainer()
                }
            }
            post {
                success {
                    script {
                        cleanUpImages()
                    }
                }
            }
        }
        stage('Check Logs') {
            steps {
                script {
                    checkDockerLogs()
                }
            }
        }
    }
}

def printInfo() {
    sh(
        script: '''
        whoami
        pwd
        echo "Hostname: $(hostname)"
        echo "REGISTRY_PROJECT: $REGISTRY_PROJECT"
        echo "REGISTRY_URL: $REGISTRY_URL"
        echo "DOCKER_IMAGE: $DOCKER_IMAGE"
        echo "DOCKER_CONTAINER: $DOCKER_CONTAINER"
        ''',
        label: 'Show info Jenkins Agent'
    )
}

def dockerLogin() {
    withCredentials([usernamePassword(
        credentialsId: 'docker_registry_creds',
        usernameVariable: 'REGISTRY_USER',
        passwordVariable: 'REGISTRY_PASSWD')]) {
        sh(
            script: 'echo "$REGISTRY_PASSWD" | docker login $REGISTRY_URL -u $REGISTRY_USER --password-stdin',
            label: 'Login to Docker'
        )
        }
}

def buildAndPushDockerImage() {
    sh(script: 'docker build -t ${DOCKER_IMAGE} ./src/demo', label: 'Build docker image')
    sh(script: 'docker push ${DOCKER_IMAGE}', label: 'Push image')
}

def deployDockerContainer() {
    try {
        timeout(time: 30, unit: 'MINUTES') {
            env.useChoice = input(
                message: 'Can it be deployed?',
                parameters: [choice(
                    name: 'deploy',
                    choices: 'no\nyes',
                    description: 'Choose "yes" if you want to deploy'
                )]
            )
        }
        if (env.useChoice == 'yes') {
            dockerLogin()
            sh(script: 'docker pull ${DOCKER_IMAGE}', label: 'Pull docker image')
            cleanUpContainer()
            sh(script: 'docker run --name ${DOCKER_CONTAINER} -d -p 8083:8081 ${DOCKER_IMAGE}', label: 'Run docker container')
        } else {
            echo 'Deployment not confirmed. Stopping progress!'
        }
    } catch (Exception ex) {
        echo "An error occurred: ${ex.getMessage()}"
    }
}

def checkDockerLogs() {
    sh(script: 'docker logs ${DOCKER_CONTAINER}', label: 'View logs')
}

def cleanUpContainer() {
    sh(script: 'docker rm -f ${DOCKER_CONTAINER} || true', label: 'Remove old container')
}
def cleanUpImages() {
    sh(script: 'docker images --format "{{.Repository}}:{{.Tag}}" | grep "^${DOCKER_IMAGE_NAME}:" | grep -v ":${DOCKER_TAG}$" | xargs -r docker rmi -f', label: 'Remove old images')
}
```
