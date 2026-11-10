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
    }

    post {
        success {
            echo 'SUCCESS: BlogReact Docker CI pipeline completed.'
        }

        failure {
            echo 'FAILED: Check the Jenkins console output.'
        }

        always {
            echo 'BlogReact pipeline execution finished.'
        }
    }
}
