
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'
        DOCKER_IMAGE = 'cithit/roseaw'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/tewelljd-cloud/225-lab3-5.git'
        KUBECONFIG = credentials('roseaw-225')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: "${GITHUB_URL}"]]
                ])
            }
        }

        stage('Lint HTML') {
            steps {
                sh 'npm install htmlhint --save-dev'
                sh 'npx htmlhint *.html'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build(
                        "${DOCKER_IMAGE}:${IMAGE_TAG}",
                        "-f Dockerfile.build ."
                    )
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry(
                        'https://index.docker.io/v1/',
                        "${DOCKER_CREDENTIALS_ID}"
                    ) {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-dev.yaml"
                sh 'kubectl apply -f deployment-dev.yaml'
            }
        }

        stage('Deploy to Prod Environment') {
            steps {
                sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-prod.yaml"
                sh 'kubectl apply -f deployment-prod.yaml'
            }
        }

        stage('Check Kubernetes Cluster') {
            steps {
                sh 'kubectl get all'
                sh 'kubectl get ingress -o wide'
                sh 'kubectl get nodes -o wide'
                sh 'kubectl rollout status deployment/dev-deployment --timeout=120s'
                sh 'kubectl rollout status deployment/prod-deployment --timeout=120s'
                sh 'kubectl get services'
            }
        }
    }

    post {
        success {
            slackSend color: 'good',
                message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        unstable {
            slackSend color: 'warning',
                message: "Build Unstable: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        failure {
            slackSend color: 'danger',
                message: "Build Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
    }
}
