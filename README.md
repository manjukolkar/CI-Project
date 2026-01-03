
# CI Project

1️ Launch EC2 Instance

Type: t2.medium

OS: Amazon Linux 2023

Open ports:

22 (SSH)

8080 (Jenkins)

9000 (SonarQube)

2️ Install System Packages
sudo yum update -y
sudo yum install git unzip wget -y
3️ Install Docker
sudo yum install docker -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker jenkins
4️ Install Jenkins
sudo wget -O /etc/yum.repos.d/jenkins.repo \
https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins

Access Jenkins → http://<EC2-IP>:8080
Retrieve password:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Install suggested plugins and create an admin user.

5️ Install Node.js
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs
node -v
npm -v
6️ Install SonarScanner CLI (manual fix)

Official binaries were blocked; installed via Maven Central mirror.

cd /opt
sudo wget https://repo1.maven.org/maven2/org/sonarsource/scanner/cli/sonar-scanner-cli/4.6.2.2472/sonar-scanner-cli-4.6.2.2472-linux.zip -O sonar-scanner.zip
sudo unzip sonar-scanner.zip
sudo mv sonar-scanner-4.6.2.2472-linux sonar-scanner
sudo chmod -R 755 /opt/sonar-scanner
sudo ln -s /opt/sonar-scanner/bin/sonar-scanner /usr/local/bin/sonar-scanner
sonar-scanner -v
7️ Run SonarQube in Docker
sudo docker run -d --name sonar \
  -p 9000:9000 \
  sonarqube:lts-community

Access SonarQube → http://<EC2-IP>:9000
Default login:

Username: admin

Password: admin

Then generate a project token for Jenkins:

My Account → Security → Generate Token → "SONAR_TOKEN"
