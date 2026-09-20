pipeline {

    agent any

    environment {
        IMAGE_NAME = "manjulakadari/nature-app"
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                     url: 'https://github.com/Manjulakadari123/nature-.git'
    }
}
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('app') {
                    sh 'npm install'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'echo "Building Nature Application..."'
                sh 'test -f app/server.js'
            }
        }

        stage('Test') {
            steps {
                dir('app') {
                    sh 'node --check server.js'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USER" \
                        --password-stdin

                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sed "s|IMAGE_PLACEHOLDER|${IMAGE_NAME}:${IMAGE_TAG}|g" \
                    k8s/deployment-template.yaml > k8s/deployment-generated.yaml

                    kubectl apply -f k8s/deployment-generated.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    kubectl rollout status deployment/nature-app --timeout=120s
                    kubectl get pods
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed!'
        }
    }
}
