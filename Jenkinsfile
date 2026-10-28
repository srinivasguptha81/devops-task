pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *')
    }

    environment {
        IMAGE_NAME = 'srinivas0810/devops-task'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
                sh 'docker tag $IMAGE_NAME:$BUILD_NUMBER $IMAGE_NAME:latest'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh 'docker push $IMAGE_NAME:$BUILD_NUMBER'
                    sh 'docker push $IMAGE_NAME:latest'
                    sh 'docker logout'
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                    docker pull $IMAGE_NAME:latest
                    docker rm -f html-app || true
                    docker run -d \
                        --name html-app \
                        --restart unless-stopped \
                        -p 80:80 \
                        $IMAGE_NAME:latest
                '''
            }
        }
    }
}