pipeline {
    agent any

    options {
        // Disable the default automatic SCM checkout
        skipDefaultCheckout(true)
    }

    environment {
        // Use the internal SonarQube service URL
        SONARQUBE_URL = 'http://sonarqube:9000'
        SONARQUBE_TOKEN = credentials('SonarQube Authentication Token')  // Add SonarQube token in Jenkins credentials
        GITHUB_TOKEN = credentials('github-token')
    }

    stages {
        stage('Checkout') {
            steps {
                sh '''
                if [ -d "flask-App" ]; then
                    rm -r flask-App
                fi
                git clone https://github.com/ImaneHoubbane99/flask-App
                '''
            }
        }

        stage('Setup Virtual Environment') {
            steps {
                script {
                    docker.image('python:3.8-slim').inside('-u root') {
                        sh '''
                        apt-get update
                        apt-get install -y python3-venv

                        if [ ! -d "venv" ]; then
                            python3 -m venv venv
                        fi

                        . venv/bin/activate
                        pip install -r flask-App/requirements.txt
                        '''
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    docker.image('python:3.8-slim').inside('-u root') {
                        sh '''
                        . venv/bin/activate
                        pytest flask-App
                        '''
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    docker.image('python:3.8-slim').inside('-u root') {
                        withSonarQubeEnv('SonarQube') {
                            sh '''
                            sonar-scanner \
                              -Dsonar.projectKey=flask-App \
                              -Dsonar.sources=flask-App \
                              -Dsonar.host.url=$SONARQUBE_URL \
                              -Dsonar.login=$SONARQUBE_TOKEN
                            '''
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Trigger Github Actions") {
            steps {
                script {
                    docker.image('python:3.8-slim').inside('-u root') {
                        echo "Trigger Github Actions"
                        sh '''
                        curl -X POST \
                        -H "Authorization: token $GITHUB_TOKEN" \
                        -H "Accept: application/vnd.github.everest-preview+json" \
                        https://api.github.com/repos/ImaneHoubbane99/deployment-files/dispatches \
                        -d '{"event_type":"trigger-workflow"}'
                        '''
                    }
                }
            }
        }
    }
}
