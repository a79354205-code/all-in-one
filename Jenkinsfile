pipeline {
    agent any
    
    environment {
        IMAGE_NAME = 'abdelrahman12345648484/flask-hello'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('2. Repository Checksum') {
            steps {
                script {
                    sh 'git rev-parse HEAD > repo_checksum.txt'
                    echo "Repository Checksum created successfully."
                }
            }
        }

        stage('3. Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('4. Docker Hub Login & Tag/Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", 
                                                  usernameVariable: 'DOCKER_USER', 
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                        sh "docker tag ${IMAGE_NAME}:latest ${IMAGE_NAME}:${BUILD_NUMBER}"
                        sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('5. Run Docker Image') {
            steps {
                script {
                    sh "docker stop my-running-app || true"
                    sh "docker rm my-running-app || true"
                    sh "docker run -d --name my-running-app -p 3001:5000 ${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution finished."
        }
    }
}
