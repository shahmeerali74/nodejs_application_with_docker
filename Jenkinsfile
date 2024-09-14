pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    credentialsId: 'github-credential',
                    url: "https://github.com/shahmeerali74/nodejs_application_with_docker.git"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "node_application:latest"

                    // Check if the Docker image already existss
                    def imageExists = sh(script: "docker images -q ${imageName}", returnStdout: true).trim()

                    if (imageExists) {
                        echo "Docker image '${imageName}' already exists. Skipping build."
                    } else {
                        echo "Docker image '${imageName}' does not exist. Building the image."
                        // Build the Docker image
                        sh "docker build -t ${imageName} ."
                    }
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    def imageName = "node_application:latest"
                    def dockerHubRepo = "shahmeerali/nodejs_application_with_docker" // Replace with your Docker Hub repo name

                    // Log in to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'Docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                    }

                    // Tag the image for Docker Hub
                    sh "docker tag ${imageName} ${dockerHubRepo}:latest"

                    // Push the image to Docker Hub
                    sh "docker push ${dockerHubRepo}:latest"
                }
            }
        }
    }
}
