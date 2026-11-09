pipeline {
    agent {
        label 'windows-docker'
    }

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
                bat '''
                    if not exist backend\\package.json exit /b 1
                    if not exist backend\\package-lock.json exit /b 1
                    if not exist backend\\Dockerfile exit /b 1
                    if not exist frontend\\package.json exit /b 1
                    if not exist frontend\\package-lock.json exit /b 1
                    if not exist frontend\\Dockerfile exit /b 1
                    if not exist docker-compose.yml exit /b 1
                    echo All required project files are present.
                '''
            }
        }

        stage('Verify Docker') {
            steps {
                bat '''
                    docker --version
                    docker info --format "{{.OSType}}"
                '''
            }
        }

        stage('Build Backend Image') {
            steps {
                bat 'docker build -t blogreact-backend:ci ./backend'
            }
        }

        stage('Build Frontend Image') {
            steps {
                bat 'docker build -t blogreact-frontend:ci ./frontend'
            }
        }

        stage('Verify Docker Images') {
            steps {
                bat '''
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