pipeline {
    agent {
        docker {
            image 'docker:24-dind' // Docker-in-Docker image
            args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        IMAGE_NAME = "jewelry-site"
        TAG = "latest"
        DOCKERHUB_CREDENTIALS = "dockerhub-creds" // Jenkins credentials ID
        DOCKERHUB_USERNAME = "your-dockerhub-username"
        DOCKERHUB_REPO = "your-dockerhub-username/jewelry-site"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/Vasu-dhara-12/jewelary.git', branch: 'main'
            }
        }

        stage('Debug Workspace') {
            steps {
                sh 'pwd'
                sh 'ls -l'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${IMAGE_NAME}:${TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                        echo $PASSWORD | docker login -u $USERNAME --password-stdin
                        docker tag ${IMAGE_NAME}:${TAG} ${DOCKERHUB_REPO}:${TAG}
                        docker push ${DOCKERHUB_REPO}:${TAG}
                    """
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                echo 'Running Docker container...'
                sh "docker run -d -p 8080:80 ${IMAGE_NAME}:${TAG}"
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Succeeded!'
        }
        failure {
            echo '❌ Deployment Failed!'
        }
    }
}
