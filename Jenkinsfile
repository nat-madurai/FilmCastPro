pipeline {
    agent any

    environment {
        // Adjust for your registry
        DOCKER_REGISTRY = "docker.io"
        DOCKER_REPO     = "visshnu12345nat/jenkins-server" // Replace with your Docker hub Username and repo
        DOCKER_IMAGE_TAG = "${env.BUILD_NUMBER}"
        // SonarQube config name in Jenkins
        // SONARQUBE_ENV = "MySonarQubeServer"
        // Teams Webhook credential ID (Secret Text)
        TEAMS_WEBHOOK_ID = 'teams-webhook'
    }

    tools {
        // If you installed NodeJS plugin in Jenkins, name must match Tools config
        nodejs "Node18"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Create Docker Image') {
            steps {
                script {
                    docker.build(
                        "${DOCKER_REPO}:${DOCKER_IMAGE_TAG}",
                        "-f Dockerfile ."
                    )
                }
            }
        }

        stage('Push Docker Image to Registry') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin $DOCKER_REGISTRY
                            docker push ${DOCKER_REPO}:${DOCKER_IMAGE_TAG}
                        """
                    }
                }
            }
        }
    }
}
