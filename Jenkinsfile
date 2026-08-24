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
                sh 'node --version'
                sh 'npm --version'
                sh 'git --version'
                sh 'docker --version'
                sh 'docker-compose --version'
            }
        }

        stage('Install dependencies') {
            steps {
                dir('backend') {
                    sh 'npm ci'
                }

                dir('frontend') {
                    sh 'npm ci'
                }
            }
        }

        stage('Lint') {
            steps {
                dir('backend') {
                    sh 'npm run lint'
                }

                dir('frontend') {
                    sh 'npm run lint'
                }
            }
        }

        stage('Build Docker images') {
            steps {
                script {
                    env.IMAGE_TAG = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                }

                sh 'docker-compose build'
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

                    sh '''
                        echo "$DOCKERHUB_TOKEN" | docker login \
                            --username "$DOCKERHUB_LOGIN" \
                            --password-stdin
                    '''

                    sh 'docker-compose push'

                    sh '''
                        docker tag "$DOCKERHUB_USERNAME/jerney-backend:$IMAGE_TAG" \
                                   "$DOCKERHUB_USERNAME/jerney-backend:latest"

                        docker tag "$DOCKERHUB_USERNAME/jerney-frontend:$IMAGE_TAG" \
                                   "$DOCKERHUB_USERNAME/jerney-frontend:latest"
                    '''

                    sh '''
                        docker push "$DOCKERHUB_USERNAME/jerney-backend:latest"
                        docker push "$DOCKERHUB_USERNAME/jerney-frontend:latest"
                    '''

                    sh 'docker logout'
                }
            }
        }
    }

    post {
        success {
            echo 'CI pipeline completed successfully.'
            echo 'Docker images were built and pushed to Docker Hub.'
        }

        failure {
            echo 'Pipeline failed. Check the failed stage console output.'
        }
    }
}