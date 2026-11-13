
pipeline {
    agent any

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

        stage('Build Application') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Verify Docker') {
            steps {
                sh 'docker info'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t kanban-dashboard:jenkins-build .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        set +x

                        echo "$DOCKER_TOKEN" | docker login \
                          -u "$DOCKER_USER" --password-stdin

                        docker tag kanban-dashboard:jenkins-build \
                          sanjay243/kanban-dashboard:latest

                        docker push sanjay243/kanban-dashboard:latest

                        docker logout
                    '''
                }
            }
        }
    }
}
