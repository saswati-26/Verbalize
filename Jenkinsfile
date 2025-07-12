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
                script{
                    echo "Building Docker image using docker-compose..."
                    echo "Image will be named: ${IMAGE_NAME}:${IMAGE_TAG}"
                    if (isUnix()) {
                        sh 'docker-compose -f docker.compose.yaml build'
                    } else {
                        bat 'docker-compose -f docker.compose.yaml build'
                    }
                    echo "Image build complete."
                }
            }
        }

        // create and start container in detach mode (-d)
        stage('3. Docker Compose Up') {
            steps {
                script {
                    echo "Creating and Running the docker container in detach mode"
                    if (isUnix()) {
                        sh 'docker-compose -f docker.compose.yaml up -d'
                    } else {
                        bat 'docker-compose -f docker.compose.yaml up -d'
                    }
                }
            }
        }

        // login to DockerHub
        stage('4. Login to Docker Hub') {
            steps {
                echo 'Logging into Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        if (isUnix()) {
                            sh "docker login -u '${DOCKER_USER}' -p '${DOCKER_PASS}'"
                        } else {
                            bat "docker login -u '${DOCKER_USER}' -p '${DOCKER_PASS}'"
                        }
                    }
                }
                echo 'Login successful.'
            }
        }

        // push the newly built Docker image to DockerHub repo
        stage('5. Push Docker Image') {
            steps {
                script {
                    echo "Pushing image ${IMAGE_NAME}:${IMAGE_TAG} to Docker Hub..."
                    if (isUnix()) {
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    } else {                    
                        bat "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                    
                    echo "Tagging image as 'latest'..."
                    if (isUnix()) {
                        sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                    } else {                    
                        bat "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                    }
                    
                    echo "Pushing 'latest' tag to Docker Hub..."
                    if (isUnix()) {
                        sh "docker push ${IMAGE_NAME}:latest"
                    } else {                    
                        bat "docker push ${IMAGE_NAME}:latest"
                    }
                }
                echo "Push complete."
            }
        }

        stage('6. Deploy Application') {
            steps {
                script {
                    echo "Deploying image ${IMAGE_NAME}:${IMAGE_TAG}..."

                    withCredentials([
                        string(credentialsId: 'mongodb-uri', variable: 'MONGODB_CONNECTION_STRING'),
                        string(credentialsId: 'jwt-secret', variable: 'JWT_SECRET_KEY')
                    ]) {

                        bat "docker stop verbalize-app || true"
                        bat "docker rm verbalize-app || true"

                        bat """
                            docker run -d --name verbalize-app -p 5000:5000 ^
                            -e MONGODB_URI="${MONGODB_CONNECTION_STRING}" ^
                            -e JWT_SECRET="${JWT_SECRET_KEY}" ^
                            -e PORT=5000 ^
                            ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }    
    }

    // post-build actions
    post {
        always {
            script {
                echo 'Pipeline finished. Logging out from Docker Hub for security.'
                if (isUnix()) {
                    sh 'docker logout'
                } else {                    
                    bat 'docker logout'
                }
            }
        }
        failure {
            echo 'Pipeline failed due to some reason. Check logs.'
        }
    }
}