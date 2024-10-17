YOUR_GITHUB_TOKEN
username/repo

1-Set Up Jenkins on VM 1:
1.1 Install Java
sudo apt update
sudo apt install openjdk-11-jdk
1.2 Add Jenkins Repository and Install Jenkins:
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins
1.3 Start Jenkins:
sudo systemctl start jenkins
sudo systemctl enable jenkins
1.4 
http://<Jenkins-VM-IP>:8080

2-Set Up SonarQube on VM 2
2.1 Install Java
sudo apt update
sudo apt install openjdk-11-jdk
2.2 Install SonarQube:
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.7.0.61563.zip
sudo apt install unzip
unzip sonarqube-9.7.0.61563.zip
sudo mv sonarqube-9.7.0.61563 /opt/sonarqube
2.3 Create a User and Set Permissions:
sudo groupadd sonar
sudo useradd -d /opt/sonarqube -g sonar sonar
sudo chown -R sonar:sonar /opt/sonarqube
2.4 Run SonarQube
sudo su - sonar
/opt/sonarqube/bin/linux-x86-64/sonar.sh start
2.5 access:
http://<SonarQube-VM-IP>:9000 admin:admin

3-Install and Configure SonarScanner on Jenkins VM (VM 1):
3.1 Download and Install SonarScanner:
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.7.0.2747-linux.zip
unzip sonar-scanner-cli-4.7.0.2747-linux.zip
sudo mv sonar-scanner-4.7.0.2747-linux /opt/sonarscanner

3.2 Configure Environment Variables:
echo 'export PATH=$PATH:/opt/sonarscanner/bin' >> ~/.bashrc
source ~/.bashrc

3.3 Test SonarScanner: 
sonar-scanner --version


4-Connect Jenkins with SonarQube:
Install the SonarQube Plugin in Jenkins:

Go to Manage Jenkins -> Manage Plugins -> Available.
Search for SonarQube Scanner and install it.
Configure SonarQube in Jenkins:

Go to Manage Jenkins -> Configure System.
Scroll down to SonarQube Servers.
Add a new SonarQube server with:
Name: SonarQube
Server URL: http://<SonarQube-VM-IP>:9000
Server authentication token:
Generate a token in SonarQube:
Log into SonarQube -> My Account -> Security -> Generate Tokens.
Use this token in Jenkins.
Configure SonarScanner in Jenkins:

Go to Manage Jenkins -> Global Tool Configuration.
Scroll to SonarQube Scanner.
Click Add SonarQube Scanner and specify the installation directory /opt/sonarscanner.





