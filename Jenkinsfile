pipeline {
    agent none  // No default agent; we specify per stage

    triggers {
        pollSCM('* * * * *')
    }

    environment {
        IMAGE_NAME = "todo-app"
        IMAGE_TAG = "v1.0.${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = credentials('docker123')
        DOCKER_HUB_REPO = "yashwanthm998/todo-app"
        K8S_TOKEN = credentials('k8s-cred')
        K8S_SERVER = "https://34.46.170.140"
    }

    stages {
        stage('Checkout') {
            agent any  // Can be your Jenkins master
            steps {
                echo("Pulling code from GitHub repository")
                checkout scm
            }
        }

        stage('Build Docker Image') {
            agent {
                label 'docker-agent'  // Your Docker-enabled agent
            }
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Login to DockerHub') {
            agent {
                label 'docker-agent'
            }
            steps {
                sh """
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                """
            }
        }

        stage('Tag & Push Docker Image') {
            agent {
                label 'docker-agent'
            }
            steps {
                sh """
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                    docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_REPO}:latest
                    docker push ${DOCKER_HUB_REPO}:latest
                """
            }
        }

        stage('Deploy to Kubernetes') {
            agent {
                kubernetes {
                    label 'kubernetes'
                    yaml """
                        apiVersion: v1
                        kind: Pod
                        spec:
                        containers:
                        - name: kubectl
                            image: bitnami/kubectl:latest
                            command:
                            - cat
                            tty: true
                        """
                }
            }
            steps {
                withCredentials([string(credentialsId: 'k8s-cred', variable: 'K8S_TOKEN')]) {
                    sh """
                        kubectl config set-cluster my-cluster --server=$K8S_SERVER --insecure-skip-tls-verify=true
                        kubectl config set-credentials jenkins --token=$K8S_TOKEN
                        kubectl config set-context my-context --cluster=my-cluster --user=jenkins
                        kubectl config use-context my-context

                        kubectl apply -f deployment.yaml --validate=false
                        kubectl set image deployment/todo-app todo-app=${DOCKER_HUB_REPO}:${IMAGE_TAG} --record
                        kubectl rollout status deployment/todo-app
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
