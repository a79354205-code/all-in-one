pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = 'abdelrahman12345648484/flask-hello'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker build \
                        --tag "${IMAGE_NAME}:build-${BUILD_NUMBER}" \
                        --tag "${IMAGE_NAME}:latest" \
                        .
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run --rm "${IMAGE_NAME}:build-${BUILD_NUMBER}" \
                        python -c "from hello import app; r = app.test_client().get('/'); assert r.status_code == 200; assert r.data == b'Hello, World!'"
                '''
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login \
                            --username "$DOCKER_USER" \
                            --password-stdin

                        docker push "${IMAGE_NAME}:build-${BUILD_NUMBER}"
                        docker push "${IMAGE_NAME}:latest"
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo "Published ${IMAGE_NAME}:build-${BUILD_NUMBER} and ${IMAGE_NAME}:latest"
        }
    }
}
