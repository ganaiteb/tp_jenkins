pipeline {
    agent any
    options { timestamps() }
    
    environment {
        IMAGE = 'itebbb/monapp' 
        TAG = "build-${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps { 
                checkout scm 
            }
        }
        
        stage('Docker Build') {
            steps {
                bat 'docker version'
                bat "docker build -t %IMAGE%:%TAG% ."
            }
        }
        
        stage('Smoke Test') {
            steps {
                bat """
                    docker rm -f monapp_test 2>nul || ver > nul
                    docker run -d --name monapp_test -p 8084:80 %IMAGE%:%TAG%
                    ping -n 4 127.0.0.1 > nul
                    curl -s http://localhost:8084 | find "Bienvenue"
                    docker rm -f monapp_test
                """
            }
        }
        
        stage('Push (Docker Hub)') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub-creds',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
            bat '''
                powershell -NoProfile -NonInteractive -Command "$env:DOCKER_PASS | docker login -u $env:DOCKER_USER --password-stdin"

                if errorlevel 1 exit /b 1

                docker tag %IMAGE%:%TAG% %IMAGE%:latest

                docker push %IMAGE%:%TAG%

                if errorlevel 1 exit /b 1

                docker push %IMAGE%:latest
            '''
        }
    }
}
    
  post {
        success {
            echo 'Build+Test+Push OK'
        }
        failure {
            echo 'Build/Test+Push KO'
        }
    }
}
