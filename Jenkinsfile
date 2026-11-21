
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'pranaovrj/kanban-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        APP_HOST = '172.31.37.185'
    }

    parameters {
        booleanParam(
            name: 'SIMULATE_HEALTH_FAILURE',
            defaultValue: false,
            description: 'Test rollback by simulating a health-check failure'
        )
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

        stage('Capture Previous Image') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'app-ec2-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    script {
                        env.PREVIOUS_IMAGE = sh(
                            script: '''
                                ssh -i "$SSH_KEY" \
                                -o BatchMode=yes \
                                -o StrictHostKeyChecking=accept-new \
                                "$SSH_USER@$APP_HOST" \
                                "docker inspect kanban-test \
                                --format '{{.Config.Image}}'"
                            ''',
                            returnStdout: true
                        ).trim()

                        if (!env.PREVIOUS_IMAGE) {
                            error('Could not identify previous image. Deployment stopped.')
                        }

                        echo "Previous image: ${env.PREVIOUS_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy to Application EC2') {
            steps {
                script {
                    env.DEPLOYMENT_ATTEMPTED = 'true'
                }

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
                        "$SSH_USER@$APP_HOST" \
                        "set -e
                         docker pull ${IMAGE_NAME}:${IMAGE_TAG}
                         docker rm -f kanban-test || true
                         docker run -d \
                            --name kanban-test \
                            --restart unless-stopped \
                            -p 8080:8080 \
                            ${IMAGE_NAME}:${IMAGE_TAG}
                        "
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                script {
                    if (params.SIMULATE_HEALTH_FAILURE) {
                        error('Simulated health-check failure for rollback testing')
                    }
                }

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
                        "$SSH_USER@$APP_HOST" \
                        "set -e
                         for attempt in 1 2 3 4 5 6 7 8 9 10; do
                            if curl -fsS http://localhost:8080/ > /dev/null; then
                                echo 'Health check passed!'
                                exit 0
                            fi
                            sleep 3
                         done
                         echo 'Health check failed!'
                         exit 1
                        "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully! Application is healthy.'
        }

        failure {
            echo 'Pipeline failed. Checking whether rollback is required.'

            script {
                if (env.DEPLOYMENT_ATTEMPTED == 'true' &&
                    env.PREVIOUS_IMAGE?.trim()) {

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
                            "$SSH_USER@$APP_HOST" \
                            "set -e
                             docker pull '$PREVIOUS_IMAGE'
                             docker rm -f kanban-test || true
                             docker run -d \
                                --name kanban-test \
                                --restart unless-stopped \
                                -p 8080:8080 \
                                '$PREVIOUS_IMAGE'

                             for attempt in 1 2 3 4 5 6 7 8 9 10; do
                                if curl -fsS http://localhost:8080/ > /dev/null; then
                                    echo 'Rollback successful! Previous application restored.'
                                    exit 0
                                fi
                                sleep 3
                             done

                             echo 'Rollback health check failed!'
                             exit 1
                            "
                        '''
                    }
                } else {
                    echo 'Rollback skipped. Deployment was not attempted or previous image is unavailable.'
                }
            }
        }
    }
}



