pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "kirthiksubbiah/my-jenkins-app"
        DOCKER_TAG = "latest"
        DOCKER_REPO = "kirthiksubbiah/my-jenkins-app"
        DOCKER_CREDENTIALS_ID = "93c470a0-e8fe-425c-8f55-932aae8919d4"
        CONTAINER_NAME = "mycontainer17"
        CONTAINER_NAME1 = "mycontainer18"
    }
    stages {
        stage('Clone Repository') {
            steps {
                echo "Cloning repository..."
                git url: 'https://github.com/kirthiksubbiah/Jenkins-Docker.git', branch: 'main'
                echo "Repository cloned successfully."
            }
        }
        
        stage('Docker Login') {
            steps {
                script {
                    echo "Logging into Docker Hub..."
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        echo "Logged into Docker Hub"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    echo "Docker image built successfully."
                }
            }
        }

        stage('Run Container Locally') {
            steps {
                script {
                    echo "Stopping and removing any existing containers..."
                    sh """
                        docker ps -q --filter name=${CONTAINER_NAME} | xargs -r docker stop || true
                        docker ps -q --filter name=${CONTAINER_NAME} | xargs -r docker rm || true
                    """
                    echo "Running new container locally..."
                    sh "docker run -d -p 8098:80 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    echo "Container running locally."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "Pushing Docker image to Docker Hub..."
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_REPO}:${DOCKER_TAG}"
                        sh "docker push ${DOCKER_REPO}:${DOCKER_TAG}"
                    }
                    echo "Docker image pushed successfully."
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    echo "Deploying Docker image to remote server..."
                    sh """
                    sshpass -p 'root' ssh -o StrictHostKeyChecking=no master@192.168.203.128 \\
                    'docker pull ${DOCKER_REPO}:${DOCKER_TAG} && \\
                    docker ps -q --filter name=${CONTAINER_NAME1} | xargs -r docker stop || true && \\
                    docker ps -q --filter name=${CONTAINER_NAME1} | xargs -r docker rm || true && \\
                    docker run -d -p 80:80 --name ${CONTAINER_NAME1} ${DOCKER_REPO}:${DOCKER_TAG}'
                    """
                    echo "Deployment completed."
                    }
                }
            }

    }
}
