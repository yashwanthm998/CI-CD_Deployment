pipeline {
    agent any

    environment {
        IMAGE_NAME = "todo-app"
        IMAGE_TAG = "v1.0.${BUILD_NUMBER}" 
        DOCKERHUB_CREDENTIALS = credentials('docker123')
        DOCKER_HUB_REPO = "yashwanthm998/todo-app"
        K8S_TOKEN = credentials('k8s-cred')
        K8S_SERVER = "https://35.239.231.236"
    }

    stages {
        stage('Checkout') {
            steps {
                echo("Pulling code from GitHub repository")
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                script {
                    echo " Logging into Docker Hub..."
                    sh """
                        echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    """
                }
            }
        }
 
        stage('Tag & Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker image to DockerHub..."
                    sh """
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                        docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_REPO}:latest
                        docker push ${DOCKER_HUB_REPO}:latest
                    """
                }
            }
        }
        stage('Deployment to kubernetes'){
            steps {
                script {
                    sh """
                        kubectl config set-cluster my-cluster --server=$K8S_SERVER --insecure-skip-tls-verify=true
                        kubectl config set-credentials jenkins --token=$K8S_TOKEN
                        kubectl config set-context my-context --cluster=my-cluster --user=jenkins
                        kubectl config use-context my-context

                        kubectl apply -f dep
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Please check the logs for details."
        }
    }
}
