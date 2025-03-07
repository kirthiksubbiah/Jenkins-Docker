pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "my-jenkins-app"
        DOCKER_TAG = "latest"
        DOCKER_REPO = "kirthiksubbiah/my-jenkins-app"
        DOCKER_CREDENTIALS_ID = "93c470a0-e8fe-425c-8f55-932aae8919d4"
        CONTAINER_NAME = "mycontainer15"
        CONTAINER_NAME1 = "mycontainer16"
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/kirthiksubbiah/Jenkins-Docker.git'
            }
        }
        stage('Docker Login') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        echo "Logged into Docker Hub"
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }
        stage('Run Container Locally') {
            steps {
                script {
                    sh """
                        # Stop and remove existing container if running
                        docker ps -q --filter name=${CONTAINER_NAME} | xargs -r docker stop || true
                        docker ps -q --filter name=${CONTAINER_NAME} | xargs -r docker rm || true

                        # Run new container
                        docker run -d -p 8097:80 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_REPO}:${DOCKER_TAG}"
                        sh "docker push ${DOCKER_REPO}:${DOCKER_TAG}"
                    }
                }
            }
        }
        stage('Deploy to Server') {
            steps {
                script {
                    sh """
                        sshpass -p "root" ssh -o StrictHostKeyChecking=no master@192.168.203.128 '
                        docker pull ${DOCKER_REPO}:${DOCKER_TAG} &&
                        docker ps -q --filter name=${CONTAINER_NAME1} | xargs -r docker stop || true &&
                        docker ps -q --filter name=${CONTAINER_NAME1} | xargs -r docker rm || true &&
                        docker run -d -p 80:80 --name ${CONTAINER_NAME1} ${DOCKER_REPO}:${DOCKER_TAG}'
                    """
                }
            }
        }
    }
}
