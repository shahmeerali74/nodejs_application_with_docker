pipeline {
    agent any

    environment {
        VERSION_FILE = 'version.txt'
        SONAR_SCANNER_HOME = tool 'SonarScanner'
        sonar_token="${env.sonarqube_token}"
        sonar_url="${env.sonar_url}"
        SLACK_CHANNEL = 'devops-jenkins-node-app'
        SLACK_CREDENTIALS_ID = 'slack-credentials-id' // Define this in Jenkins' credentials
        
    }

    stages {
        stage('Checkout') {
            steps {
                slackSend(channel: "${SLACK_CHANNEL}", message: "Checkout started", tokenCredentialId: "${SLACK_CREDENTIALS_ID}")
                git branch: 'master',
                    credentialsId: 'github-credential',
                    url: "https://github.com/shahmeerali74/nodejs_application_with_docker.git"
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    slackSend(channel: "${SLACK_CHANNEL}", message: "SonarQube scan started", tokenCredentialId: "${SLACK_CREDENTIALS_ID}")
                    withSonarQubeEnv('Sonarqube') {
                        sh """
                            ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=nodejs_application_with_docker \
                            -Dsonar.sources=. \
                            -Dsonar.language=js \
                            -Dsonar.host.url=${sonar_url} \
                            -Dsonar.login=${sonar_token}
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
                    slackSend(channel: "${SLACK_CHANNEL}", message: "Building Docker image with version ${env.VERSION}", tokenCredentialId: "${SLACK_CREDENTIALS_ID}")
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "nodeapp${env.VERSION}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    def dockerHubRepo = "shahmeerali/nodejs_application_with_docker"
                    def imageName = "nodeapp${env.VERSION}"

                    withCredentials([usernamePassword(credentialsId: 'Docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                    }

                    sh "docker tag ${imageName} ${dockerHubRepo}:${imageName}"
                    sh "docker push ${dockerHubRepo}:${imageName}"

                    sh "docker rmi -f nodeapp:latest || true"
                }
            }
        }

        stage('Clean Up Old Docker Images') {
            steps {
                script {
                    def dockerHubRepo = "shahmeerali/nodejs_application_with_docker"
                    sh "docker images --format \"{{.Repository}}:{{.Tag}}\" | grep '^nodeapp' | grep -v ':${env.VERSION}' | xargs -r docker rmi -f || true"
                }
            }
        }

        stage('Deploy New Container') {
            steps {
                script {
                    def imageName = "shahmeerali/nodejs_application_with_docker:nodeapp${env.VERSION}"
                    def containerName = "nodeapp"

                    sh """
                        docker stop ${containerName} || true
                        docker rm ${containerName} || true
                    """

                    sh "docker run -d --name ${containerName} -p 3000:3000 ${imageName}"

                    slackSend(channel: "${SLACK_CHANNEL}", message: "Deployed new container with version ${env.VERSION}", tokenCredentialId: "${SLACK_CREDENTIALS_ID}")
                }
            }
        }

        stage('Update Version File') {
            steps {
                script {
                    writeFile file: VERSION_FILE, text: "${env.VERSION}"
                    slackSend(channel: "${SLACK_CHANNEL}", message: "Version file updated to version ${env.VERSION}", tokenCredentialId: "${SLACK_CREDENTIALS_ID}")
                }
            }
        }
    }

    post {
        success {
            slackSend(channel: "${SLACK_CHANNEL}", message: "Pipeline completed successfully with version ${env.VERSION}", tokenCredentialId: "${SLACK_CREDENTIALS_ID}")
        }
        failure {
            slackSend(channel: "${SLACK_CHANNEL}", message: "Pipeline failed", tokenCredentialId: "${SLACK_CREDENTIALS_ID}")
        }
    }
}
