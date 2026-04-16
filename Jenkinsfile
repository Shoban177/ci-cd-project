pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'shoban177/cicd-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'pip3 install -r requirements.txt --break-system-packages'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running tests...'
                sh '/var/jenkins_home/.local/bin/pytest test_app.py -v'
            }
        }

        stage('Code Quality - SonarQube') {
            steps {
                echo 'Skipping SonarQube - will configure separately...'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t shobanababu/cicd-app:${BUILD_NUMBER} ."
                sh "docker tag shobanababu/cicd-app:${BUILD_NUMBER} shobanababu/cicd-app:latest"
            }
        }

        stage('Docker Push') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub-creds',
            usernameVariable: 'USER',
            passwordVariable: 'PASS'
        )]) {
            sh '''
                echo "$PASS" | docker login -u "$USER" --password-stdin
                docker push shobanababu/cicd-app:${BUILD_NUMBER}
                docker push shobanababu/cicd-app:latest
            '''
        }
    }
}

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh "docker stop cicd-app || true"
                sh "docker rm cicd-app || true"
                sh "docker run -d --name cicd-app -p 5000:5000 ${DOCKERHUB_REPO}:latest"
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline SUCCESS!'
        }
        failure {
            echo '❌ Pipeline FAILED!'
        }
    }
}