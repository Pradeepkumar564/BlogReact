pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        DOCKER_BUILDKIT = '1'
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

        stage('Install and Verify Backend') {
            steps {
                sh '''
                    set -eu
                    docker run --rm \
                      -v "$WORKSPACE/backend:/app" \
                      -w /app \
                      node:22-alpine \
                      npm ci --omit=dev --no-audit --no-fund
                '''
            }
        }

        stage('Build Frontend') {
            steps {
                sh '''
                    set -eu
                    docker run --rm \
                      -v "$WORKSPACE/frontend:/app" \
                      -w /app \
                      node:22-alpine \
                      sh -c "npm ci --no-audit --no-fund && npm run build"
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    set -eu
                    docker build -t blogreact-backend:ci ./backend
                    docker build -t blogreact-frontend:ci ./frontend
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: BlogReact CI pipeline completed.'
        }
        failure {
            echo 'FAILED: Check the stage logs to identify the problem.'
        }
        always {
            echo 'BlogReact CI pipeline execution finished.'
        }
    }
}
