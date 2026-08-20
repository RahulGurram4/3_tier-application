pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'gurramrahul'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Check tools') {
            steps {
                bat 'node --version'
                bat 'npm --version'
                bat 'docker --version'
                bat 'docker compose version'
            }
        }

        stage('Install dependencies') {
            steps {
                dir('backend') {
                    bat 'npm ci'
                }
                dir('frontend') {
                    bat 'npm ci'
                }
            }
        }

        stage('Lint') {
            steps {
                dir('backend') {
                    bat 'npm run lint'
                }
                dir('frontend') {
                    bat 'npm run lint'
                }
            }
        }

        stage('Build Docker images') {
            steps {
                script {
                    env.IMAGE_TAG = bat(
                        returnStdout: true,
                        script: '@git rev-parse --short HEAD'
                    ).trim()
                }
                bat 'docker compose build'
            }
        }

        stage('Push images to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKERHUB_LOGIN',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    bat 'powershell -NoProfile -Command "$env:DOCKERHUB_TOKEN | docker login --username $env:DOCKERHUB_LOGIN --password-stdin"'
                    bat 'docker compose push'
                    bat 'docker tag %DOCKERHUB_USERNAME%/jerney-backend:%IMAGE_TAG% %DOCKERHUB_USERNAME%/jerney-backend:latest'
                    bat 'docker tag %DOCKERHUB_USERNAME%/jerney-frontend:%IMAGE_TAG% %DOCKERHUB_USERNAME%/jerney-frontend:latest'
                    bat 'docker push %DOCKERHUB_USERNAME%/jerney-backend:latest'
                    bat 'docker push %DOCKERHUB_USERNAME%/jerney-frontend:latest'
                    bat 'docker logout'
                }
            }
        }
    }

    post {
        success {
            echo 'Images built and pushed to Docker Hub successfully.'
        }
        failure {
            echo 'Pipeline failed. Open the failed stage log for details.'
        }
    }
}