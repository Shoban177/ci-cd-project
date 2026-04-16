pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'your-dockerhub-username/cicd-app'
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
                sh 'pytest test_app.py -v'
            }
        }

        stage('Code Quality - SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=cicd-app \
                        -Dsonar.sources=. \
                        -Dsonar.python.version=3.11
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} ."
                sh "docker tag ${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_REPO}:latest"
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_REPO}:latest"
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