
pipeline {
    agent any

    options {
        disableConcurrentBuilds()
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
                        set -eu
                        set +x

                        export DOCKER_CONFIG="$(mktemp -d)"
                        trap 'rm -rf "$DOCKER_CONFIG"' EXIT

                        echo "$DOCKER_TOKEN" | docker login \
                          -u "$DOCKER_USER" --password-stdin

                        docker tag kanban-dashboard:jenkins-build \
                          sanjay243/kanban-dashboard:latest

                        docker push sanjay243/kanban-dashboard:latest
                    '''
                }
            }
        }

        stage('Deploy to EC2 Test') {
            steps {
                sh '''
                    set -eu

                    IMAGE=sanjay243/kanban-dashboard:latest

                    docker pull "$IMAGE"

                    # Remove the backup from an earlier deployment
                    docker rm -f kanban-test-previous 2>/dev/null || true

                    # Preserve the currently running test container
                    if docker container inspect kanban-test >/dev/null 2>&1; then
                        docker stop kanban-test
                        docker rename kanban-test kanban-test-previous
                    fi

                    rollback() {
                        echo "Deployment failed. Rolling back."

                        docker rm -f kanban-test 2>/dev/null || true

                        if docker container inspect kanban-test-previous >/dev/null 2>&1; then
                            docker rename kanban-test-previous kanban-test
                            docker start kanban-test
                        fi
                    }

                    trap rollback EXIT

                    docker run -d \
                      --name kanban-test \
                      --restart unless-stopped \
                      --memory 256m \
                      --cpus 0.5 \
                      -p 8082:8080 \
                      "$IMAGE"

                    echo "Waiting for container health..."

                    HEALTH=unknown

                    for i in $(seq 1 30); do
                        HEALTH=$(docker inspect \
                          --format '{{.State.Health.Status}}' \
                          kanban-test)

                        echo "Health status: $HEALTH"

                        if [ "$HEALTH" = "healthy" ]; then
                            break
                        fi

                        if [ "$HEALTH" = "unhealthy" ]; then
                            break
                        fi

                        sleep 2
                    done

                    if [ "$HEALTH" != "healthy" ]; then
                        echo "Container did not become healthy."
                        exit 1
                    fi

                    curl --fail --silent --show-error \
                      http://localhost:8082/ > /dev/null

                    echo "Deployment and HTTP validation successful."

                    trap - EXIT
                '''
            }
        }
    }
}
