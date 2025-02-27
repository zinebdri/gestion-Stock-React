pipeline {
    agent any
    environment {
        IMAGE_NAME = 'zineb932/gestion-stock-react'
        IMAGE_TAG = 'latest'
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
                        sh 'docker build --no-cache -t $IMAGE_NAME:$IMAGE_TAG .'
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
                            sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
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
                        sh 'kubectl apply -f k8s/deployment.yml'
                        sh 'kubectl rollout restart deployment gestion-stock-react'
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
                        sh 'docker --version'
                        sh 'kubectl version --short'
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
            // Vous pouvez ajouter ici des étapes pour nettoyer ou archiver des artefacts si nécessaire
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
    }
}
