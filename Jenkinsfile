pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'alsdevops/als'
        DOCKER_HUB_CREDENTIALS = '4d645895-93f6-4365-9dc4-a1cdd7ccff09' // Credentials ID in Jenkins
        REMOTE_SERVER = 'ubuntu@34.194.252.73' // Remote server SSH credentials
        SSH_CREDENTIALS_ID = '43de5b36-781f-4d25-8b49-b525406e69f3' // SSH Credentials ID in Jenkins
        CONTAINER_NAME = 'als'
        APP_PORT = '1337'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from GitHub
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Get the short commit hash
                    COMMIT_HASH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    
                    // Build Docker image
                    sh 'docker build -t ${DOCKER_HUB_REPO}:latest .'
                    sh 'docker tag ${DOCKER_HUB_REPO}:latest ${DOCKER_HUB_REPO}:${COMMIT_HASH}'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_HUB_CREDENTIALS) {
                        sh 'docker push ${DOCKER_HUB_REPO}:latest'
                        sh 'docker push ${DOCKER_HUB_REPO}:${COMMIT_HASH}'
                    }
                }
            }
        }
        stage('Run Tests') {
            steps {
                // Example test command, replace with your actual test commands
                sh 'docker run --rm ${DOCKER_HUB_REPO}:latest npm test'
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // SSH into the remote server and deploy the new Docker container
                    sshagent([SSH_CREDENTIALS_ID]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_SERVER} << EOF
                            docker pull ${DOCKER_HUB_REPO}:latest
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true
                            docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:${APP_PORT} ${DOCKER_HUB_REPO}:latest
                        EOF
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up
            sh 'docker rmi ${DOCKER_HUB_REPO}:latest'
            sh 'docker rmi ${DOCKER_HUB_REPO}:${COMMIT_HASH}'
        }
    }
}
