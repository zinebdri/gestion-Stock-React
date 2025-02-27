pipeline {
    agent any
    
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/zinebdri/gestion-Stock-React.git'
            }
        }

        stage('Build docker image') {
            steps {
                script {
                    sh 'docker build -t zineb932/gestion-stock-react:latest .'
                }
            }
        }
    }
}
