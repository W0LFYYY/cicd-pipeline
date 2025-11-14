pipeline {
    agent any
    
    environment {
        WEBEX_BOT_TOKEN = credentials('webex-bot-token')
        WEBEX_ROOM_ID = 'PLACEHOLDER_ROOM_ID'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code from GitHub...'
                checkout scm
            }
        }
        
        stage('Setup Environment') {
            steps {
                echo 'Setting up Python environment...'
                sh '''
                    python3 --version
                    pip3 --version
                '''
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh '''
                    pip3 install --user -r requirements.txt
                '''
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building the project...'
                sh 'chmod +x build.sh'
                sh './build.sh'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh 'chmod +x test.sh'
                sh './test.sh'
            }
        }
    }
    
    post {
        success {
            echo 'Build succeeded! Sending notification to WebEx...'
            script {
                sh """
                    curl -X POST 'https://webexapis.com/v1/messages' \
                    -H 'Authorization: Bearer ${WEBEX_BOT_TOKEN}' \
                    -H 'Content-Type: application/json' \
                    -d '{
                        "roomId": "${WEBEX_ROOM_ID}",
                        "markdown": "**BUILD SUCCESS**\\n\\n**Job:** ${env.JOB_NAME}\\n**Build:** #${env.BUILD_NUMBER}\\n**Status:** All tests passed\\n\\n[View Build](${env.BUILD_URL})"
                    }'
                """
            }
        }
        
        failure {
            echo 'Build failed! Sending notification to WebEx...'
            script {
                sh """
                    curl -X POST 'https://webexapis.com/v1/messages' \
                    -H 'Authorization: Bearer ${WEBEX_BOT_TOKEN}' \
                    -H 'Content-Type: application/json' \
                    -d '{
                        "roomId": "${WEBEX_ROOM_ID}",
                        "markdown": "**BUILD FAILED**\\n\\n**Job:** ${env.JOB_NAME}\\n**Build:** #${env.BUILD_NUMBER}\\n**Status:** Build or tests failed\\n\\n[View Build](${env.BUILD_URL})"
                    }'
                """
            }
        }
    }
}
