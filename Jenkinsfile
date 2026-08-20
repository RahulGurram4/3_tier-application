pipeline {
    agent any

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

        stage('Validate Docker Compose') {
            steps {
                bat 'docker compose config'
            }
        }

        stage('Build Docker images') {
            steps {
                bat 'docker compose build'
            }
        }
    }

    post {
        success {
            echo 'Build completed successfully.'
        }
        failure {
            echo 'Build failed. Check the stage log above.'
        }
    }
}