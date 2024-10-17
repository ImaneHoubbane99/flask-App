YOUR_GITHUB_TOKEN
username/repo

1- install kuberntes cluster
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version
minikube start --driver=docker
minikube status
sudo apt-get update
sudo apt-get install -y kubectl
kubectl get nodes

2- install jenkins
kubectl create namespace jenkins

apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-pv
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data_jenkins/jenkins"  # Path on the local node where the data will be stored

kubectl apply -f jenkins-pv.yaml 

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pvc
  namespace: jenkins
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  selector:
    matchLabels:
      type: local

kubectl apply -f jenkins-pvc.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins/jenkins:lts
        ports:
        - containerPort: 8080
        - containerPort: 50000
        volumeMounts:
        - name: jenkins-storage
          mountPath: /var/jenkins_home  # Jenkins data directory
      volumes:
      - name: jenkins-storage
        persistentVolumeClaim:
          claimName: jenkins-pvc  # Bind the PVC to the deployment
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: jenkins
spec:
  type: NodePort  # Use NodePort for local access
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30000  # Expose Jenkins UI on localhost:30000
  - port: 50000
    targetPort: 50000
  selector:
    app: jenkins

kubectl apply -f jenkins-deployment.yaml

kubectl get pods -n jenkins

kubectl get svc -n jenkins

kubectl port-forward jenkins_pod_name -n jenkins 8080:8080

external-ip:8080

3-install sonarscanner

apiVersion: v1
kind: Pod
metadata:
  name: sonarscanner
  labels:
    app: sonarscanner
spec:
  containers:
  - name: sonarscanner
    image: sonarsource/sonar-scanner-cli:latest
    command: [ "sleep", "infinity" ]

kubectl apply -f sonarscanner-pod.yaml


kubectl exec -it sonarscanner -- /bin/bash






4-install sonarqube

apiVersion: apps/v1
kind: Deployment
metadata:
  name: sonarqube
  labels:
    app: sonarqube
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sonarqube
  template:
    metadata:
      labels:
        app: sonarqube
    spec:
      containers:
      - name: sonarqube
        image: sonarqube:latest
        ports:
        - containerPort: 9000
        volumeMounts:
        - name: sonarqube-data
          mountPath: /opt/sonarqube/data
        - name: sonarqube-logs
          mountPath: /opt/sonarqube/logs
      volumes:
      - name: sonarqube-data
        emptyDir: {}
      - name: sonarqube-logs
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: sonarqube
spec:
  type: NodePort
  ports:
  - port: 9000
    targetPort: 9000
    nodePort: 30001
  selector:
    app: sonarqube


kubectl apply -f sonarqube-deployment.yaml
kubectl get svc sonarqube : admin:admin
kubectl port-forward sonarqube-74777577d4-64zf4 9000:9000

5-Configure Jenkins to Use SonarQube

5.1 Install SonarQube Scanner Plugin
In Jenkins, install the SonarQube Scanner plugin:
Go to Manage Jenkins > Manage Plugins > Available.
Search for SonarQube Scanner and install it.

5.2 Configure SonarQube Server in Jenkins
Go to Manage Jenkins > Configure System.
Scroll down to the SonarQube servers section.
Add a new SonarQube server configuration:
Name: SonarQube
Server URL: http://sonarqube:9000 (internal DNS name of the SonarQube service).
Server Authentication Token: Add your SonarQube token as credentials.

password : Oualid1998@oualid

5.3 Configure SonarScanner in Jenkins
Go to Manage Jenkins > Global Tool Configuration.
Scroll down to SonarQube Scanner.
Add SonarQube Scanner and give it a name (e.g., SonarScanner).

