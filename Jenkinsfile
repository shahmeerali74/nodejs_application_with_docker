pipeline {
    agent any

    environment {
        VERSION_FILE = 'version.txt'
        SONAR_SCANNER_HOME = tool 'SonarScanner'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    credentialsId: 'github-credential',
                    url: "https://github.com/shahmeerali74/nodejs_application_with_docker.git"
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    withSonarQubeEnv('Sonarqube') {
                        sh """
                            ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=nodejs_application_with_docker \
                            -Dsonar.sources=. \
                            -Dsonar.language=js \
                            -Dsonar.host.url=http://practice-vm-shahmeer.tail9d8d3f.ts.net:9000/ \
                            -Dsonar.login=sqp_59f7c5dd4f17c02254ab29df9547292f88b7ec5e
                        """
                    }
                }
            }
        }

        stage('Get Current Version') {
            steps {
                script {
                    if (fileExists(VERSION_FILE)) {
                        def currentVersion = readFile(VERSION_FILE).trim()
                        env.VERSION = (currentVersion.toInteger() + 1).toString()
                    } else {
                        env.VERSION = '1'
                    }
                    echo "Building Docker image with version ${env.VERSION}."
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "nodeapp${env.VERSION}"

                    // Build the Docker image with the new version
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    def dockerHubRepo = "shahmeerali/nodejs_application_with_docker"
                    def imageName = "nodeapp${env.VERSION}"

                    // Log in to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'Docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                    }

                    // Tag the image for Docker Hub
                    sh "docker tag ${imageName} ${dockerHubRepo}:${imageName}"

                    // Push the image to Docker Hub
                    sh "docker push ${dockerHubRepo}:${imageName}"

                    // Remove the local image with the `latest` tag if it exists
                    sh """
                        docker rmi -f nodeapp:latest || true
                    """
                }
            }
        }

        stage('Clean Up Old Docker Images') {
            steps {
                script {
                    def dockerHubRepo = "shahmeerali/nodejs_application_with_docker"

                    // Clean up old images in Docker Hub (not tagged with the latest version)
                    sh """
                        docker images --format "{{.Repository}}:{{.Tag}}" | grep '^nodeapp' | grep -v ':${env.VERSION}' | xargs -r docker rmi -f || true
                    """
                }
            }
        }

        stage('Deploy New Container') {
    steps {
        script {
            def imageName = "shahmeerali/nodejs_application_with_docker:nodeapp${env.VERSION}"
            def containerName = "nodeapp"

            // Stop and remove the currently running container if it exists
            sh """
                docker stop ${containerName} || true
                docker rm ${containerName} || true
            """

            // Run the new container with the versioned image
            sh """
                docker run -d --name ${containerName} -p 3000:3000 ${imageName}
            """
        }
    }
}


        stage('Update Version File') {
            steps {
                script {
                    // Update the version file with the new version number
                    writeFile file: VERSION_FILE, text: "${env.VERSION}"
                }
            }
        }
    }
}
