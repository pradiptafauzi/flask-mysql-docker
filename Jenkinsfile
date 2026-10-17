
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'pradipta211/flask-mysql-docker'
        REGISTRY_CREDENTIALS = 'dockerhub-credentials'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Docker Compose') {
            steps {
                bat 'docker compose config -q'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %IMAGE_NAME%:%BUILD_NUMBER% ./app'
            }
        }

        stage('Login Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: REGISTRY_CREDENTIALS,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    powershell '''
                        $ErrorActionPreference = "Stop"
                        $env:DOCKER_PASS |
                            docker login -u $env:DOCKER_USER --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            throw "Login Docker Hub gagal."
                        }
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat '''
                    docker push %IMAGE_NAME%:%BUILD_NUMBER%
                    if errorlevel 1 exit /b 1

                    docker tag %IMAGE_NAME%:%BUILD_NUMBER% %IMAGE_NAME%:latest
                    if errorlevel 1 exit /b 1

                    docker push %IMAGE_NAME%:latest
                    if errorlevel 1 exit /b 1
                '''
            }
        }
    }

    post {
        always {
            bat 'docker logout'
            echo 'Pipeline selesai dijalankan.'
        }
    }
}
