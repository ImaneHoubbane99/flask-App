pipeline {
    agent any

    options {
        // Disable the default automatic SCM checkout
        skipDefaultCheckout(true)
    }

    environment {
        GITHUB_TOKEN = credentials('github-token')  // Add GitHub token in Jenkins credentials
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
                sh '''
                sudo apt-get update
                sudo apt-get install -y python3-venv

                if [ ! -d "venv" ]; then
                    python3 -m venv venv
                fi

                . venv/bin/activate
                pip install -r flask-App/requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                . venv/bin/activate
                pytest flask-App
                '''
            }
        }


        stage('Trigger GitHub Actions') {
            steps {
                echo "Triggering GitHub Actions"
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
