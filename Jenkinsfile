pipeline {
    agent any

    environment {
        DOCKER_REGISTRY   = "docker.io"
        DOCKER_REPO       = "visshnu12345nat/jenkins-server"
        DOCKER_IMAGE_TAG  = "${env.BUILD_NUMBER}"

        SONARQUBE_ENV     = "MySonarQubeServer"
        TEAMS_WEBHOOK_ID  = 'teams-webhook'
    }

    tools {
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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=jenkins-react-app \
                        -Dsonar.projectName=JenkinsReactApp \
                        -Dsonar.sources=src \
                        -Dsonar.host.url=http://13.211.175.170:9000 \
                        -Dsonar.token=$SONAR_AUTH_TOKEN
                    '''
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

        stage('Trivy Scan') {
            steps {
                sh """
                    trivy image --exit-code 0 --severity LOW,MEDIUM ${DOCKER_REPO}:${DOCKER_IMAGE_TAG}
                    trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_REPO}:${DOCKER_IMAGE_TAG}
                """
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
