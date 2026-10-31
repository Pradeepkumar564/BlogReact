```groovy
pipeline {
    agent any

    tools {
        nodejs 'NodeJS-22'
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

        stage('Install Backend Dependencies') {
            steps {
                dir('backend') {
                    sh '''
                        set -eu
                        node --version
                        npm --version
                        npm ci --omit=dev --no-audit --no-fund
                    '''
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh '''
                        set -eu
                        npm ci --no-audit --no-fund
                        npm run build
                    '''
                }
            }
        }

        stage('Archive Frontend Build') {
            steps {
                archiveArtifacts artifacts: 'frontend/dist/**',
                                 fingerprint: true
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
```