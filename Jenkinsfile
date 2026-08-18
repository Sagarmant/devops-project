pipeline {
    agent any

    environment {
        IMAGE_NAME = 'vifgt/devops-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sagarmant/devops-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build --network=host -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl set image deployment/devops-app \
                    devops-app=${IMAGE_NAME}:${IMAGE_TAG}

                    kubectl rollout status deployment/devops-app
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get pods
                    kubectl get services
                '''
            }
        }
    }
}
