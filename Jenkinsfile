pipeline {
    agent any
    stages {
        stage('checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Abhishek-aivonic/FilmCastPro.git'
                //
            }
        }
        stage('build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
                //
            }
      }
         stage('Build Multi-Platform Docker Image') {
            steps {
                script {
                    // Create a new builder instance (optional, but good practice for persistent builders)
                    
                    sh 'cd filmcastro'
                    // Build the image for multiple platforms
                    sh 'docker buildx build -t visnu/your-app:latest .' 
                    // The --push flag is essential to push the multi-platform image manifest
                }
            }
        }
    }
}
