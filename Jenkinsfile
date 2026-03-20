pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "docker.io"
        DOCKER_REPO     = "vasudhara12"
        IMAGE_NAME      = "jewelry-site"
        IMAGE_TAG       = "latest"

        CONTAINER_NAME  = "jewelry-container"
        PORT            = "8877"
        CONTAINER_PORT  = "80"
    }

    stages {

        stage('Debug Workspace') {
            steps {
                echo "Checking workspace..."
                sh "pwd"
                sh "ls -l"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image from repo root..."

                sh '''
                if [ ! -f Dockerfile ]; then
                    echo "❌ Error: Dockerfile not found!"
                    exit 1
                fi
                '''

                sh "docker build -t $IMAGE_NAME:$IMAGE_TAG ."
                sh "docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKER_REGISTRY/$DOCKER_REPO/$IMAGE_NAME:$IMAGE_TAG"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Logging in and pushing image..."

                withCredentials([usernamePassword(
                    credentialsId: 'DOCKER_CREDS',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo "$DOCKER_PASS" | docker login $DOCKER_REGISTRY -u "$DOCKER_USER" --password-stdin

                    docker push $DOCKER_REGISTRY/$DOCKER_REPO/$IMAGE_NAME:$IMAGE_TAG

                    docker logout $DOCKER_REGISTRY
                    '''
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                echo "Stopping old container (if exists)..."

                sh "docker stop $CONTAINER_NAME || true"
                sh "docker rm $CONTAINER_NAME || true"

                echo "Running new container..."

                sh """
                docker run -d \
                -p $PORT:$CONTAINER_PORT \
                --name $CONTAINER_NAME \
                $DOCKER_REGISTRY/$DOCKER_REPO/$IMAGE_NAME:$IMAGE_TAG
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed!"
        }
    }
}
