pipeline {
    agent any

    environment {
        IMAGE_NAME = 'pranaovrj/kanban-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t ${IMAGE_NAME}:${IMAGE_TAG} \
                    -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Login and Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_PASSWORD'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKERHUB_PASSWORD" | docker login \
                        -u "$DOCKERHUB_USERNAME" --password-stdin

                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Test SSH Connection') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'app-ec2-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        ssh -i "$SSH_KEY" \
                            -o BatchMode=yes \
                            -o StrictHostKeyChecking=accept-new \
                            -o ConnectTimeout=10 \
                            "$SSH_USER@172.31.37.185" \
                            "hostname && whoami"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}


