pipeline {
    agent {
        docker {
            image 'abhishekf5/maven-abhishek-docker-agent:v1'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    
    options {
        // Disable the default automatic SCM checkout
        skipDefaultCheckout(true)
    }

    stages {
        stage('Install Docker') {
            steps {
                sh '''
                # Check if Docker is installed
                if ! [ -x "$(command -v docker)" ]; then
                    echo "Docker is not installed. Installing Docker..."

                    # Update package list and install necessary packages
                    apt-get update

                    apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release

                    # Add Docker’s official GPG key
                    curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

                    # Set up the Docker stable repository for Debian
                    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

                    # Install Docker
                    apt-get update
                    apt-get install -y docker-ce docker-ce-cli containerd.io

                    echo "Docker installed successfully."
                else
                    echo "Docker is already installed."
                fi
                '''
            }
        }

        stage('Checkout') {
            steps {
                sh '''
                # Check if the flask-App directory is empty or does not exist, and clone if necessary
                if [  -d "flask-App" ]; then
                    rm -r flask-App
                fi
                echo "Cloning flask-App repository"
                git clone https://github.com/ImaneHoubbane99/flask-App
                '''
            }
        }

        stage('Setup Virtual Environment') {
            steps {
                sh '''
                # Install venv if not already installed
                apt-get update
                apt-get install -y python3-venv

                # Create a virtual environment if it doesn't exist
                if [ ! -d "venv" ]; then
                    python3 -m venv venv
                fi

                # Activate the virtual environment
                . venv/bin/activate

                # Install dependencies in the virtual environment
                pip install -r flask-App/requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                # Activate the virtual environment for running tests
                . venv/bin/activate

                # Run your tests
                pytest flask-App
                '''
            }
        }

        stage("Login to Docker Hub") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin' // login safely
                }
                echo 'Login successfully'
            }
        }

        stage('Build Docker Image') {
            environment {
                IMAGE_NAME = 'imane123456788/flask-app'
                IMAGE_TAG = "${IMAGE_NAME}:v${env.BUILD_ID}"
            }
            steps {
                sh 'docker build -t ${IMAGE_TAG} flask-App'
                echo "Docker image built successfully"
                sh 'docker image ls'
            }
        }

        stage("Push to Docker Hub") {
            environment {
                IMAGE_NAME = 'imane123456788/flask-app'
                IMAGE_TAG = "${IMAGE_NAME}:v${env.BUILD_ID}"
            }
            steps {
                sh 'docker push ${IMAGE_TAG}'
                echo "Docker image pushed successfully"
            }
        }

        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "deployment-files" // replace with your GitHub repository name
                GIT_USER_NAME = "ImaneHoubbane99" // replace with your GitHub account username
            }
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "ihoubbane@gmail.com" // replace with your GitHub user email
                        git config user.name "ImaneHoubbane99" // replace with your GitHub username
                        OLD_BUILD_NUMBER=$((${BUILD_NUMBER}-1))
                        sed -i "s/${OLD_BUILD_NUMBER}/${BUILD_NUMBER}/g" flask-App/k8s/deployment.yml
                        cd flask-App/k8s
                        git add deployment.yml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
    }
}
