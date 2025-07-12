// declarative pipeline

pipeline {

    agent any
    environment {
        DOCKERHUB_USERNAME = 'saswati26'
        IMAGE_NAME = "${DOCKERHUB_USERNAME}/verbalize"
        IMAGE_TAG = "v${env.BUILD_NUMBER}" 
    }

    stages {
        // Stage 1: Checkout the source code from your Git repository
        stage('1. Checkout Code') {
            steps {
                echo 'Checking out the latest source code from GitHub...'
                checkout scm
            }
        }

        // build the Docker image using the Dockerfile
        stage('2. Build Docker Image') {
            steps {
                echo "Building Docker image using docker-compose..."
                echo "Image will be named: ${IMAGE_NAME}:${IMAGE_TAG}"
                sh 'docker-compose -f docker.compose.yaml build'
                echo "Image build complete."
            }
        }

        // create and start container in detach mode (-d)
        stage('3. Docker Compose Up') {
            steps {
                echo "Creating and Running the docker container in detach mode"
                sh 'docker-compose -f docker.compose.yaml up -d'
            }
        }

        // login to DockerHub
        stage('4. Login to Docker Hub') {
            steps {
                echo 'Logging into Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker login -u '${DOCKER_USER}' -p '${DOCKER_PASS}'"
                }
                echo 'Login successful.'
            }
        }

        // push the newly built Docker image to DockerHub repo
        stage('5. Push Docker Image') {
            steps {
                echo "Pushing image ${IMAGE_NAME}:${IMAGE_TAG} to Docker Hub..."
                sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                
                echo "Tagging image as 'latest'..."
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                
                echo "Pushing 'latest' tag to Docker Hub..."
                sh "docker push ${IMAGE_NAME}:latest"
                
                echo "Push complete."
            }
        }
    }

    // post-build actions
    post {
        always {
            echo 'Pipeline finished. Logging out from Docker Hub for security.'
            sh 'docker logout'
        }
        failure {
            echo 'Pipeline failed due to some reason. Check logs.'
        }
    }
}