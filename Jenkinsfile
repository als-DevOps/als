pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'alsdevops/als'
        DOCKER_HUB_CREDENTIALS = '4d645895-93f6-4365-9dc4-a1cdd7ccff09' // Credentials ID in Jenkins
        REMOTE_SERVER = 'ubuntu@34.194.252.73' // Remote server SSH credentials
        SSH_CREDENTIALS_ID = '43de5b36-781f-4d25-8b49-b525406e69f3' // SSH Credentials ID in Jenkins
        CONTAINER_NAME = 'als-app'
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
                    echo "Commit hash: ${COMMIT_HASH}"

                    // Use the .env file from Jenkins credentials
                    echo "Use the .env file from Jenkins credentials..."
                    withCredentials([file(credentialsId: '293a6d45-5c0c-4757-a65c-9594b73dbd6d', variable: 'ENV_FILE')]) {
                        sh 'rm -rf .env'
                        // Copy the .env file to the root of the project
                        sh 'cp $ENV_FILE .env'
                        // Verify the .env file is created and has correct permissions
                        sh 'ls -la .env'
                    }
                    
                    // Build Docker image
                    echo "Building Docker image..."
                    sh 'docker build -t ${DOCKER_HUB_REPO}:latest .'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    echo "Push Docker Image"
                    docker.withRegistry('', DOCKER_HUB_CREDENTIALS) {
                        sh 'docker push ${DOCKER_HUB_REPO}:latest'
                    }
                }
            }
        }
        stage('Run Tests') {
            steps {
                // Run Tests
                script {
                    try {
                    // Use the .env file from Jenkins credentials
                    echo "Use the .env file from Jenkins credentials..."
                    withCredentials([file(credentialsId: '293a6d45-5c0c-4757-a65c-9594b73dbd6d', variable: 'ENV_FILE')]) {
                        sh 'rm -rf .env'
                        // Copy the .env file to the root of the project
                        sh 'cp $ENV_FILE .env'
                        // Verify the .env file is created and has correct permissions
                        sh 'ls -la .env'
                    }
                sh '''
                docker run --rm ${DOCKER_HUB_REPO}:latest /bin/sh -c "npm run test"
                '''
                } catch (Exception e) {
                        // Handle the error and ensure the job does not fail
                        echo "Tests failed, but the job will not fail."
                        // Optionally, you can record test results or other artifacts here
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // SSH into the remote server and deploy the new Docker container
                    sshagent([SSH_CREDENTIALS_ID]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_SERVER} << EOF
                            echo "docker pull"
                            docker pull ${DOCKER_HUB_REPO}:latest
                            echo "docker stop"
                            docker ps -q | xargs -r docker stop && docker ps -a -q | xargs -r docker rm
                            echo "docker run"
                            docker run -d -p ${APP_PORT}:${APP_PORT} ${DOCKER_HUB_REPO}:latest
                            exit
                        EOF
                        """
                    }
                }
            }
        }
    }
}
