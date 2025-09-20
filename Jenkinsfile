pipeline {
    agent any

    environment {
        DOCKER_REGISTRY  = "docker.io"
        DOCKER_REPO      = "visshnu12345nat/jenkins-server"
        DOCKER_IMAGE_TAG = "${env.BUILD_NUMBER}"
        TEAMS_WEBHOOK_ID = 'teams-webhook'
    }

    tools {
        nodejs "Node18"    // Jenkins → Global Tool Configuration
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
                withSonarQubeEnv('MySonarQubeServer') {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=my-react-app \
                          -Dsonar.sources=src \
                          -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                    '''
                }
            }
        }

        stage('Create Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REPO}:${DOCKER_IMAGE_TAG}", "-f Dockerfile .")
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    // Fails pipeline if HIGH/CRITICAL vulnerabilities are found
                    sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_REPO}:${DOCKER_IMAGE_TAG}"
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
                            docker tag ${DOCKER_REPO}:${DOCKER_IMAGE_TAG} ${DOCKER_REPO}:latest
                            docker push ${DOCKER_REPO}:latest
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG_FILE')]) {
                        sh """
                            export KUBECONFIG=$KUBECONFIG_FILE
                            kubectl set image deployment/react-deployment react-container=${DOCKER_REPO}:${DOCKER_IMAGE_TAG} -n my-namespace
                            kubectl rollout status deployment/react-deployment -n my-namespace
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline successful: Built, Scanned, and Deployed!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
