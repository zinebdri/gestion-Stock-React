pipeline {
    agent any
    environment {
        IMAGE_NAME = 'zineb932/gestion-stock-react'
        IMAGE_TAG = 'latest'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git credentialsId: 'github-credentials', branch: 'main', url: 'https://github.com/zinebdri/gestion-Stock-React.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build --no-cache -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }
        stage('Push to Docker Registry') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: 'https://index.docker.io/v1/']) {
                    sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yml'
                sh 'kubectl rollout restart deployment gestion-stock-react'
            }
        }
        stage('Verify Docker and Kubernetes') {
            steps {
                sh 'docker --version'
                sh 'kubectl version --short'
            }
        }
    }
}
