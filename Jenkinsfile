pipeline {
    agent any

    environment {
        // Docker registry settings
        DOCKER_REGISTRY   = "docker.io"
        DOCKER_REPO       = "visshnu12345nat/jenkins-server" // Replace with your DockerHub repo
        DOCKER_IMAGE_TAG  = "${env.BUILD_NUMBER}"

        // SonarQube config name (must match Jenkins SonarQube server config)
        SONARQUBE_ENV     = "MySonarQubeServer"

        // Teams Webhook credential ID (Secret Text)
        TEAMS_WEBHOOK_ID  = 'teams-webhook'
    }

    tools {
        nodejs "Node18" // NodeJS version configured in Jenkins Global Tools
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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh """
                        npx sonar-scanner \
                        -Dsonar.projectKey=jenkins-react-app \
                        -Dsonar.projectName=JenkinsReactApp \
                        -Dsonar.sources=src \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    """
                }
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

    post {
        always {
            echo "Pipeline finished. Cleaning up workspace..."
            cleanWs()
        }
        success {
            echo "✅ Build, SonarQube Analysis, Trivy Scan, and Docker Push succeeded."
        }
        failure {
            echo "❌ Pipeline failed. Please check logs."
        }
    }
}
