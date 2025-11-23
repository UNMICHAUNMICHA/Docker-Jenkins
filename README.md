
# ✅ **1. ติดตั้ง Jenkins บน Docker**

1.  สร้างโฟลเดอร์ Jenkins:
    

`mkdir -p /root/jenkins/jenkins_home chmod -R 777 /root/jenkins/jenkins_home cd /root/jenkins` 

2.  สร้างไฟล์ **docker-compose.yml**:
    

`version:  '3.8'  services:  jenkins:  image:  jenkins/jenkins:lts  container_name:  jenkins  user:  root  ports:  -  "8080:8080"  -  "50000:50000"  volumes:  -  ./jenkins_home:/var/jenkins_home  -  /var/run/docker.sock:/var/run/docker.sock  -  /usr/bin/docker:/usr/bin/docker  environment:  -  DOCKER_HOST=unix:///var/run/docker.sock  restart:  unless-stopped` 

3.  รัน Jenkins:
    

`docker compose up -d` 

4.  ดู password สำหรับ login Jenkins:
    

`cat /root/jenkins/jenkins_home/secrets/initialAdminPassword` 

หรือ

`docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword` 

----------

# ✅ **2. ทำให้ Jenkins สั่ง Docker / Compose ได้**

-   Jenkins container **mount docker.sock** และไบนารี docker → สามารถสั่ง `docker` และ `docker compose` บน host ได้
    
-   ติดตั้ง Jenkins plugin:
    
    -   Docker Pipeline
        
    -   Docker plugin
        
    -   Pipeline
        

----------

# ✅ **3. เขียน Jenkinsfile สำหรับ deploy Docker stack**

ตัวอย่าง **Jenkinsfile**:

`pipeline {
    agent any

    stages {
        stage('Check Docker') {
            steps {
                sh 'docker ps'
            }
        }

        stage('Deploy Stack') {
            steps {
                sh '''
                  cd /root/portainer
                  docker compose up -d
                '''
            }
        }
    }
}` 

**อธิบาย:**

1.  Jenkins pipeline จะสั่ง `docker ps` เช็คว่ามีสิทธิ์ใช้งาน Docker
    
2.  เข้าโฟลเดอร์ `/root/portainer` (หรือโฟลเดอร์ stack ของคุณ)
    
3.  รัน `docker compose up -d` → deploy service ทั้ง stack
    

----------

# ✅ **4. ขยายระบบ**

-   เพิ่ม stage สำหรับ build image → push registry → deploy
    
-   ใช้ webhook GitHub/GitLab → auto trigger pipeline
    
-   ตั้ง schedule / monitor container ด้วย Jenkins stage
