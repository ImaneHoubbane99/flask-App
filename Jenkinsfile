pipeline {
    agent any
    
    options {
        skipDefaultCheckout(true)  // Disable the default automatic SCM checkout
    }

    stages {
        stage('Install Docker') {
            steps {
                bat '''
                docker -v
                if %ERRORLEVEL% neq 0 (
                    echo Docker not installed. Please install Docker manually on Windows.
                ) else (
                    echo Docker is already installed.
                )
                '''
            }
        }

        stage('Checkout') {
            steps {
                bat '''
                if exist flask-App (rmdir /S /Q flask-App)
                echo Cloning flask-App repository
                git clone https://github.com/ImaneHoubbane99/flask-App
                '''
            }
        }

        stage('Setup Virtual Environment') {
            steps {
                bat '''
                python -m venv venv
                venv\\Scripts\\activate
                pip install -r flask-App\\requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                bat '''
                venv\\Scripts\\activate
                pytest flask-App
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    bat 'echo %PASSWORD% | docker login -u %USERNAME% --password-stdin'
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
                bat 'docker build -t %IMAGE_TAG% flask-App'
                echo "Docker image built successfully"
                bat 'docker images'
            }
        }

        stage('Push to Docker Hub') {
            environment {
                IMAGE_NAME = 'imane123456788/flask-app'
                IMAGE_TAG = "${IMAGE_NAME}:v${env.BUILD_ID}"
            }
            steps {
                bat 'docker push %IMAGE_TAG%'
                echo "Docker image pushed successfully"
            }
        }

        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "deployment-files" 
                GIT_USER_NAME = "ImaneHoubbane99"
            }
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    bat '''
                    git config user.email "ihoubbane@gmail.com"
                    git config user.name "ImaneHoubbane99"
                    FOR /F "tokens=1 delims=." %%i IN ('%BUILD_NUMBER%') DO (SET OLD_BUILD_NUMBER=%%i)
                    sed -i "s/%OLD_BUILD_NUMBER%/%BUILD_NUMBER%/g" flask-App\\k8s\\deployment.yml
                    cd flask-App\\k8s
                    git add deployment.yml
                    git commit -m "Update deployment image to version %BUILD_NUMBER%"
                    git push https://%GITHUB_TOKEN%@github.com/%GIT_USER_NAME%/%GIT_REPO_NAME% HEAD:main
                    '''
                }
            }
        }
    }
}
