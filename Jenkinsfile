pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'gurramrahul'
        EC2_HOST = '3.84.171.54'
        EC2_USER = 'ubuntu'
        DEPLOY_DIR = '/opt/jerney'
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

        stage('Deploy to AWS') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'aws-ec2-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {

                    sh '''
                        chmod 600 "$SSH_KEY"

                        ssh \
                          -i "$SSH_KEY" \
                          -o StrictHostKeyChecking=no \
                          -o UserKnownHostsFile=/dev/null \
                          "$SSH_USER@$EC2_HOST" \
                          "cd $DEPLOY_DIR && \
                           git fetch origin main && \
                           git reset --hard origin/main && \
                           export DOCKERHUB_USERNAME=$DOCKERHUB_USERNAME && \
                           export IMAGE_TAG=$IMAGE_TAG && \
                           docker compose pull && \
                           docker compose up -d"
                    '''
                }
            }
        }

        stage('AWS health check') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'aws-ec2-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {

                    sh '''
                        chmod 600 "$SSH_KEY"

                        ssh \
                          -i "$SSH_KEY" \
                          -o StrictHostKeyChecking=no \
                          -o UserKnownHostsFile=/dev/null \
                          "$SSH_USER@$EC2_HOST" \
                          "for i in 1 2 3 4 5 6; do \
                               if curl -fsS http://localhost:8080/api/health; then \
                                   exit 0; \
                               fi; \
                               sleep 5; \
                           done; \
                           exit 1"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
            echo 'Docker images were pushed and AWS deployment passed the health check.'
        }

        failure {
            echo 'Pipeline failed. Check the failed stage console output.'
        }
    }
}