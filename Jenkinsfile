
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
    }
}
