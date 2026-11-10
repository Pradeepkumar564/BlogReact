pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('Verify Project Files') {
            steps {
                sh '''
                    set -eu
                    test -f backend/package.json
                    test -f backend/package-lock.json
                    test -f backend/Dockerfile
                    test -f frontend/package.json
                    test -f frontend/package-lock.json
                    test -f frontend/Dockerfile
                    test -f docker-compose.yml
                    echo "All required project files are present."
                '''
            }
        }

        stage('Verify Docker') {
            steps {
                sh '''
                    docker --version
                    docker info --format "{{.OSType}}"
                '''
            }
        }

        stage('Build Backend Image') {
            steps {
                sh 'docker build -t blogreact-backend:ci ./backend'
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh 'docker build -t blogreact-frontend:ci ./frontend'
            }
        }

        stage('Verify Docker Images') {
            steps {
                sh '''
                    docker images blogreact-backend
                    docker images blogreact-frontend
                '''
            }
        }

        stage('Push Docker Images to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-docker-credentials',
                    usernameVariable: 'NEXUS_USERNAME',
                    passwordVariable: 'NEXUS_PASSWORD'
                )]) {
                    sh '''
                        set -eu
                        set +x

                        echo "$NEXUS_PASSWORD" | docker login \
                            host.docker.internal:8083 \
                            --username "$NEXUS_USERNAME" \
                            --password-stdin

                        docker tag blogreact-backend:ci \
                            host.docker.internal:8083/blogreact-backend:${BUILD_NUMBER}

                        docker tag blogreact-frontend:ci \
                            host.docker.internal:8083/blogreact-frontend:${BUILD_NUMBER}

                        docker push host.docker.internal:8083/blogreact-backend:${BUILD_NUMBER}

                        docker push host.docker.internal:8083/blogreact-frontend:${BUILD_NUMBER}

                        docker logout host.docker.internal:8083
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: BlogReact CI pipeline and Nexus image push completed.'
        }

        failure {
            echo 'FAILED: Check the Jenkins console output.'
        }

        always {
            echo 'BlogReact pipeline execution finished.'
        }
    }
}
