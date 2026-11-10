
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '478397108493'
        ECR_REGISTRY = '478397108493.dkr.ecr.ap-south-1.amazonaws.com'
        EKS_CLUSTER = 'blogreact-eks'
        K8S_NAMESPACE = 'blogreact'
        BACKEND_IMAGE = 'blogreact-backend'
        FRONTEND_IMAGE = 'blogreact-frontend'
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
                    set -eu
                    docker --version
                    docker info --format "{{.OSType}}"
                '''
            }
        }

        stage('Verify AWS Credentials') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-credentials',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        set -eu
                        set +x
                        export AWS_DEFAULT_REGION="$AWS_REGION"
                        export AWS_EC2_METADATA_DISABLED=true

                        echo "Checking AWS authentication..."
                        aws sts get-caller-identity
                        echo "AWS authentication successful."
                    '''
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                sh '''
                    set -eu
                    docker build -t blogreact-backend:ci ./backend
                '''
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh '''
                    set -eu
                    docker build -t blogreact-frontend:ci ./frontend
                '''
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

        stage('Login to Amazon ECR') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-credentials',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        set -eu
                        set +x

                        export AWS_DEFAULT_REGION="$AWS_REGION"
                        export AWS_EC2_METADATA_DISABLED=true

                        aws ecr get-login-password \
                            --region "$AWS_REGION" \
                        | docker login \
                            --username AWS \
                            --password-stdin "$ECR_REGISTRY"

                        echo "ECR login successful."
                    '''
                }
            }
        }

        stage('Push Images to Amazon ECR') {
            steps {
                sh '''
                    set -eu

                    docker tag blogreact-backend:ci \
                        "$ECR_REGISTRY/$BACKEND_IMAGE:$BUILD_NUMBER"

                    docker tag blogreact-frontend:ci \
                        "$ECR_REGISTRY/$FRONTEND_IMAGE:$BUILD_NUMBER"

                    docker push "$ECR_REGISTRY/$BACKEND_IMAGE:$BUILD_NUMBER"

                    docker push "$ECR_REGISTRY/$FRONTEND_IMAGE:$BUILD_NUMBER"

                    echo "Both images pushed to Amazon ECR."
                '''
            }
        }

        stage('Configure EKS Access') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-credentials',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        set -eu
                        set +x

                        export AWS_DEFAULT_REGION="$AWS_REGION"
                        export AWS_EC2_METADATA_DISABLED=true

                        aws eks update-kubeconfig \
                            --region "$AWS_REGION" \
                            --name "$EKS_CLUSTER"

                        kubectl get nodes

                        echo "EKS connection successful."
                    '''
                }
            }
        }

        stage('Deploy to Amazon EKS') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-credentials',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        set -eu
                        set +x

                        export AWS_DEFAULT_REGION="$AWS_REGION"
                        export AWS_EC2_METADATA_DISABLED=true

                        kubectl set image deployment/backend \
                            backend="$ECR_REGISTRY/$BACKEND_IMAGE:$BUILD_NUMBER" \
                            -n "$K8S_NAMESPACE"

                        kubectl set image deployment/frontend \
                            frontend="$ECR_REGISTRY/$FRONTEND_IMAGE:$BUILD_NUMBER" \
                            -n "$K8S_NAMESPACE"

                        echo "Application images updated in EKS."
                    '''
                }
            }
        }

        stage('Verify EKS Deployment') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-credentials',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        set -eu
                        set +x

                        export AWS_DEFAULT_REGION="$AWS_REGION"
                        export AWS_EC2_METADATA_DISABLED=true

                        echo "Checking backend rollout..."

                        kubectl rollout status deployment/backend \
                            -n "$K8S_NAMESPACE" \
                            --timeout=180s

                        echo "Checking frontend rollout..."

                        kubectl rollout status deployment/frontend \
                            -n "$K8S_NAMESPACE" \
                            --timeout=180s

                        echo "Checking application pods..."

                        kubectl get pods -n "$K8S_NAMESPACE"

                        echo "Checking application services..."

                        kubectl get services -n "$K8S_NAMESPACE"

                        echo "EKS deployment completed successfully."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: BlogReact CI/CD pipeline completed. Images pushed to Nexus and ECR, and EKS deployment verified.'
        }

        failure {
            echo 'FAILED: Check Jenkins Console Output for the failing stage.'
        }

        always {
            echo 'BlogReact pipeline execution finished.'
        }
    }
}
