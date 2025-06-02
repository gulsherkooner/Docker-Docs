Docker Environment Setup on Hetzner Cloud
This guide provides a step-by-step process to set up SSH access, configure a firewall, install Docker and Docker Compose, and deploy a microservices application on an Ubuntu 22.04 server hosted with Hetzner Cloud. The application includes auth-service, post-service, sub-service, dating-service, next-frontend, api-gateway, and shared infrastructure (nginx, postgres, redis).

🔑 1. Generate SSH Key on Windows

Open PowerShell on your Windows machine.

Generate an SSH key pair:
ssh-keygen -t ed25519 -C "Ubuntu-4gb-hel1-2"


Press Enter to accept the default file location (C:\Users\your_username\.ssh\id_ed25519).
Press Enter twice to skip setting a passphrase (or set one for extra security).


Verify the key:
Get-Content C:\Users\uset\.ssh\id_ed25519.pub


Copy the output (e.g., ssh-ed25519 AAAAC3Nza... ubuntu-4gb-hel1-2).




☁️ 2. Add SSH Key to Hetzner Cloud

Log in to the Hetzner Cloud Console.
Navigate to Project > Security > SSH Keys.
Click Add SSH Key.
Paste the public key copied in Step 1.
Name the key (e.g., ubuntu-4gb-hel1-2).
Click Add SSH Key.


🧪 3. Test SSH Access from Windows

Connect to the server:Replace 135.181.192.55 with your server’s IP.
ssh root@135.181.192.55


Type yes to trust the host key if prompted.


Expected Output:
root@cent-stage-server:~#


Update the server:
apt update && apt upgrade -y


Exit the SSH session:
exit




🔥 4. Configure Firewall

In the Hetzner Cloud Console, go to Project > Security > Firewalls.
Click Create Firewall.
Set Name: cent-stage-firewall.
Add Inbound Rules:


Service
Protocol
Port
Source IPs



SSH
TCP
22
0.0.0.0/0


HTTP
TCP
80
0.0.0.0/0


HTTPS
TCP
443
0.0.0.0/0


Jenkins
TCP
8080
0.0.0.0/0


PostgreSQL
TCP
5432
0.0.0.0/0*


Redis
TCP
6379
0.0.0.0/0*



*Restrict PostgreSQL and Redis to your server’s IP or VPN for production.


Apply to Server: Select your server (e.g., ubuntu-4gb-hel1-2).
Click Create Firewall.
On the server, enable ufw (optional):sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 8080/tcp
sudo ufw enable




🐳 5. Install Docker and Docker Compose

Connect to the server:
ssh root@135.181.192.55


Update package list:
sudo apt update


Install prerequisites:
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common


Add Docker’s GPG key:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg


Set up Docker repository:
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


Update package index:
sudo apt update


Install Docker:
sudo apt install -y docker-ce docker-ce-cli containerd.io


Verify Docker:
sudo docker --version


Enable Docker service:
sudo systemctl start docker
sudo systemctl enable docker


Install Docker Compose:
DOCKER_COMPOSE_VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/')
sudo curl -L "https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose


Verify Docker Compose:
docker-compose --version




👤 6. Enable Non-Root Docker Access (Optional)

Add user to Docker group:
sudo usermod -aG docker $USER


Apply changes:
exit
ssh root@135.181.192.55


Test Docker without sudo:
docker ps




✅ 7. Test Docker Installation

Run a test container:
docker run --rm hello-world


Expected: Hello from Docker!


Clean up:
docker image rm hello-world




📁 8. Set Up Deployment Directory

Create directory:mkdir -p /root/centstage
cd /root/centstage




🛠️ 9. Create Docker Network

Create a shared network:docker network create cent-stage-network




🗄️ 10. Deploy Infrastructure Services

Clone infrastructure repository:
cd /root/centstage
git clone https://github.com/gulsherkooner/infrastructure.git
cd infrastructure


Create docker-compose.yml:
nano docker-compose.yml

version: '3.8'
services:
  nginx:
    image: nginx:latest
    container_name: infrastructure-nginx-1
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    networks:
      - cent-stage-network
  postgres:
    image: postgres:14
    container_name: infrastructure-postgres-1
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: my_app
    networks:
      - cent-stage-network
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
  redis:
    image: redis:7
    container_name: infrastructure-redis-1
    networks:
      - cent-stage-network
networks:
  cent-stage-network:
    external: true


Create nginx.conf:
nano nginx.conf

http {
    proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;
    server {
        listen 80;
        location /api/post {
            proxy_pass http://api-gateway-api-gateway-1:2001/posts;
            proxy_cache my_cache;
            proxy_cache_valid 200 60s;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        location /api/auth {
            proxy_pass http://auth-service-auth-service-1:3001/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        location /api/subscription {
            proxy_pass http://sub-service-sub-service-1:3003/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        location /api/dating {
            proxy_pass http://dating-service-dating-service-1:3006/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        location /api {
            proxy_pass http://api-gateway-api-gateway-1:2001/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        location / {
            proxy_pass http://next-frontend-next-frontend-1:3000/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
events { }


Deploy infrastructure:
docker-compose up -d


Verify services:
docker ps -a | grep -E 'nginx|postgres|redis'


Expected: infrastructure-nginx-1, infrastructure-postgres-1, infrastructure-redis-1 running.




🚀 11. Deploy Microservices

Clone service repositories:
cd /root/centstage
git clone https://github.com/gulsherkooner/auth-service.git
git clone https://github.com/gulsherkooner/post-service.git
git clone https://github.com/gulsherkooner/sub-service.git
git clone https://github.com/gulsherkooner/dating-service.git
git clone https://github.com/gulsherkooner/next-frontend.git
git clone https://github.com/gulsherkooner/api-gateway.git


Create docker-compose.yml for each service (example for post-service):
cd /root/centstage/post-service
nano docker-compose.yml

version: '3.8'
services:
  post-service:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: post-service-post-service-1
    env_file:
      - .env
    networks:
      - cent-stage-network
networks:
  cent-stage-network:
    external: true


Create .env file:
nano .env

DATABASE_URL="postgresql://postgres:password@infrastructure-postgres-1:5432/my_app?schema=public"


Repeat for other services (auth-service, sub-service, dating-service, api-gateway, next-frontend), adjusting ports and configurations:

auth-service: Port 3001, /api/auth.
sub-service: Port 3003, /api/subscription.
dating-service: Port 3006, /api/dating.
api-gateway: Port 2001, /api.
next-frontend: Port 3000, /.


Deploy services:
cd /root/centstage/post-service
docker-compose up -d
cd /root/centstage/auth-service
docker-compose up -d
cd /root/centstage/sub-service
docker-compose up -d
cd /root/centstage/dating-service
docker-compose up -d
cd /root/centstage/api-gateway
docker-compose up -d
cd /root/centstage/next-frontend
docker-compose up -d




🛡️ 12. Secure Services

Restrict database access:

Update Hetzner firewall to limit PostgreSQL (5432) and Redis (6379) to the server’s IP or a VPN.
Example:# In Hetzner Console, update firewall rule
PostgreSQL: Source: 135.181.192.55
Redis: Source: 135.181.192.55




Set strong passwords:

Update POSTGRES_PASSWORD in infrastructure/docker-compose.yml.
Update .env files for services.




✅ 13. Test the Application

Test APIs:
curl http://135.181.192.55/api/post
curl http://135.181.192.55/api/auth
curl http://135.181.192.55/api/subscription
curl http://135.181.192.55/api/dating
curl http://135.181.192.55/


Check logs:
docker logs infrastructure-nginx-1
docker logs post-service-post-service-1
docker logs dating-service-dating-service-1




🔄 14. Automate with Jenkins Pipelines

Install Jenkins:
sudo apt install -y openjdk-17-jre
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install -y jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins


Access Jenkins:

Open http://135.181.192.55:8080.
Get initial password:sudo cat /var/lib/jenkins/secrets/initialAdminPassword




Create pipeline for infrastructure (example Jenkinsfile):
cd /root/centstage/infrastructure
nano Jenkinsfile

pipeline {
    agent any
    environment {
        HETZNER_IP = '135.181.192.55'
    }
    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/gulsherkooner/infrastructure.git', branch: 'main', credentialsId: 'github-token'
            }
        }
        stage('Deploy Infrastructure') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'hetzner-ssh', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no root@${HETZNER_IP} <<'EOF'
                        cd /root/centstage/infrastructure
                        docker-compose -f docker-compose.yml up -d
                        docker exec infrastructure-nginx-1 nginx -t
                        docker exec infrastructure-nginx-1 nginx -s reload



EOF                       '''                   }               }           }       }   }

4. **Create pipeline for `dating-service`** (example Jenkinsfile):
```bash
cd /root/centstage/dating-service
nano Jenkinsfile

pipeline {
    agent any
    environment {
        HETZNER_IP = '135.181.192.55'
        TAG        = "${env.GIT_COMMIT}"
    }
    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/gulsherkooner/dating-service.git',
                    branch: 'main',
                    credentialsId: 'github-token'
            }
        }
        stage('Build Docker Image') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'hetzner-ssh', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no -o ConnectTimeout=10 root@${HETZNER_IP} <<'EOF'
                        mkdir -p /root/centstage/dating-service
                        cd /root/centstage/dating-service
                        if [ ! -d .git ]; then
                            git clone https://github.com/gulsherkooner/dating-service.git .
                        else
                            git pull origin main
                        fi
                        docker build -t dating-service:${TAG} .
                        docker save -o /root/dating-service-${TAG}.tar dating-service:${TAG}
EOF
                    '''
                }
            }
        }
        stage('Deploy Service') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'hetzner-ssh', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no -o ConnectTimeout=10 root@${HETZNER_IP} <<'EOF'
                        cd /root/centstage/infrastructure
                        docker-compose -f docker-compose.yml up -d
                        cd /root/centstage/dating-service
                        docker load -i /root/dating-service-${TAG}.tar
                        docker-compose -f docker-compose.yml up -d
                        rm /root/dating-service-${TAG}.tar
                        docker exec infrastructure-nginx-1 nginx -s reload
EOF
                    '''
                }
            }
        }
        stage('Verify Service') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'hetzner-ssh', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no -o ConnectTimeout=10 root@${HETZNER_IP} <<'EOF'
                        if ! docker ps | grep -q dating-service-dating-service-1; then
                            echo "Dating service container not running"
                            exit 1
                        fi
EOF
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Dating microservice deployed successfully'
        }
        failure {
            echo 'Dating microservice deployment failed'
            withCredentials([sshUserPrivateKey(credentialsId: 'hetzner-ssh', keyFileVariable: 'SSH_KEY')]) {
                sh '''
                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no -o ConnectTimeout=10 root@${HETZNER_IP} <<'EOF'
                    cd /root/centstage/dating-service
                    docker-compose -f docker-compose.yml down
                    rm -f /root/dating-service-${TAG}.tar
EOF
                '''
            }
            error 'Rolled back dating microservice deployment'
        }
    }
}


Configure Jenkins pipeline:

In Jenkins, create infrastructure-deploy pipeline with the above Jenkinsfile.
Create dating-service-deploy pipeline with the dating-service Jenkinsfile.
Add github-token and hetzner-ssh credentials.
Set up GitHub webhook for each repository.


Repeat for other services (auth-service, post-service, sub-service, api-gateway, next-frontend) with similar Jenkinsfiles.



📝 Notes

Security: Restrict firewall ports and use strong passwords in production.
Scaling: Consider upgrading to a higher Hetzner plan (e.g., CPX31) for better performance.
Monitoring: Add logging/monitoring tools (e.g., Prometheus, Grafana) for production.

