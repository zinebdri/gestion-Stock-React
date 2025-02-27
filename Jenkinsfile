pipeline {
    agent any
    environment {
        IMAGE_NAME = 'zineb932/gestion-stock-react'
        IMAGE_TAG = 'latest'
        KUBE_NAMESPACE = 'default' // You can make this configurable
    }
    stages {
        stage('Checkout Code') {
            steps {
                script {
                    try {
                        git credentialsId: 'github-credentials', branch: 'main', url: 'https://github.com/zinebdri/gestion-Stock-React.git'
                    } catch (Exception e) {
                        error "Checkout failed: ${e.getMessage()}"
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        sh "docker build --no-cache -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    } catch (Exception e) {
                        error "Docker build failed: ${e.getMessage()}"
                    }
                }
            }
        }
        stage('Push to Docker Registry') {
            steps {
                script {
                    try {
                        withDockerRegistry([credentialsId: 'dockerhub-credentials', url: 'https://index.docker.io/v1/']) {
                            sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                        }
                    } catch (Exception e) {
                        error "Docker push failed: ${e.getMessage()}"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    try {
                        sh "kubectl apply -f k8s/deployment.yml -n ${KUBE_NAMESPACE}"
                        sh "kubectl rollout restart deployment gestion-stock-react -n ${KUBE_NAMESPACE}"
                        sh "kubectl rollout status deployment gestion-stock-react -n ${KUBE_NAMESPACE}"
                    } catch (Exception e) {
                        error "Kubernetes deployment failed: ${e.getMessage()}"
                    }
                }
            }
        }
        stage('Verify Docker and Kubernetes') {
            steps {
                script {
                    try {
                        sh "docker --version"
                        sh "kubectl version --short"
                    } catch (Exception e) {
                        error "Docker or Kubernetes verification failed: ${e.getMessage()}"
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Cleaning up resources after pipeline execution...'
            // Add cleanup steps here if necessary
        }
        success {
            echo 'Pipeline executed successfully!'
            // Add notification steps here (e.g., Slack, email)
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
            // Add notification steps here (e.g., Slack, email)
        }
    }
}
