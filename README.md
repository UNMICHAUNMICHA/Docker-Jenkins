Write-up: ตั้งค่า Jenkins + GitHub private repo + Docker Compose
1. ติดตั้ง Docker & Docker Compose

บน server (Debian/Ubuntu) หรือ VM:

# ติดตั้ง Docker Engine
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER   # ให้ user ปกติรัน docker ได้

# ตรวจสอบเวอร์ชัน
docker version

# ติดตั้ง Docker Compose v2
sudo curl -L "https://github.com/docker/compose/releases/download/v2.40.3/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose version


หมายเหตุ: ถ้าใช้ Docker Compose v2 (integrated), จะรันด้วย docker compose ไม่ต้อง docker-compose ก็ได้

2. ติดตั้ง Jenkins ด้วย Docker

สร้าง docker-compose.yml สำหรับ Jenkins:

version: '3.8'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    user: root
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
      - /usr/local/bin/docker-compose:/usr/local/bin/docker-compose
    environment:
      - DOCKER_HOST=unix:///var/run/docker.sock
    restart: unless-stopped


จากนั้นรัน:

docker-compose up -d


Jenkins จะรันที่ http://<your-server-ip>:8080

3. ตั้งค่า Jenkins

เข้าหน้า web Jenkins

Unlock Jenkins ด้วย password จาก /var/jenkins_home/secrets/initialAdminPassword

ติดตั้ง Suggested plugins

สร้าง admin user

4. เพิ่ม GitHub Credentials

ไปที่ Manage Jenkins → Credentials → System → Global credentials → Add Credentials

เลือก Username with password

Username: GitHub username

Password: Personal Access Token (PAT)

ID: github-token

5. สร้าง Jenkins Pipeline

ตัวอย่าง Jenkinsfile:

pipeline {
    agent any

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/UNMICHAUNMICHA/info.git',
                    credentialsId: 'github-token'
            }
        }

        stage('Build & Deploy') {
            steps {
                sh '/usr/local/bin/docker-compose up -d --build'
            }
        }
    }
}


Tips:

ใช้ credentialsId แทนใส่ token ใน URL

รัน /usr/local/bin/docker-compose เพราะบางครั้ง path default ของ Jenkins ไม่เจอ

6. ตรวจสอบ pipeline

สร้าง job แบบ pipeline และใส่ Jenkinsfile

กด Build Now

ตรวจสอบ console output

ขั้นตอน clone repo ผ่าน git fetch

ขั้นตอน deploy ผ่าน docker-compose up -d --build
