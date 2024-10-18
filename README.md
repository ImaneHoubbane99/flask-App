YOUR_GITHUB_TOKEN
username/repo

1- install sonarqube 

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





