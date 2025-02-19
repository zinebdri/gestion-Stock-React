pipeline {
    agent any

    environment {
        // Nom de l'image Docker frontend en fonction de la branche
        BRANCH_NAME = "${env.BRANCH_NAME ?: 'main'}"  // Si BRANCH_NAME n'est pas défini, on utilise 'main'
        frontendImage = "zineb932/gestion-stock-react:frontend-${BRANCH_NAME}"  // Image Docker nommée en fonction de la branche
    }

    stages {
        // Étape 1 : Récupérer le code depuis GitHub
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/zinebdri/gestion-Stock-React.git'
            }
        }

        // Étape 2 : Installer les dépendances et Build du frontend React
        stage('Build Frontend') {
            steps {
                dir('frontend') {  // Se déplacer dans le dossier frontend
                    sh 'npm install'  // Installe les dépendances
                    sh 'npm run build'  // Compile le projet React
                }
            }
        }

        // Étape 3 : Construire les images Docker
        stage('Build Docker Images') {
            steps {
                sh "docker build -t ${frontendImage} -f frontend/Dockerfile frontend/"
            }
        }

        // Étape 4 : Push to Docker Hub
        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub-connector',
                        usernameVariable: 'DOCKER_HUB_USERNAME',
                        passwordVariable: 'DOCKER_HUB_PASSWORD'
                    )
                ]) {
                    sh """
                        echo "$DOCKER_HUB_PASSWORD" | docker login -u "$DOCKER_HUB_USERNAME" --password-stdin
                        docker push ${frontendImage}
                    """
                }
            }
        }

        // Étape 5 : Déployer sur Kubernetes
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'k8s-config', variable: 'KUBECONFIG')]) {
                    sh """
                        # Mise à jour dynamique de l'image Docker dans le fichier YAML de Kubernetes
                        sed -i "s|zineb932/gestion-stock-react:frontend.*|${frontendImage}|g" k8s/deployment.yaml
                        
                        # Créer le namespace si nécessaire
                        kubectl create namespace zineb || true  # 'true' pour ne pas échouer si le namespace existe déjà

                        # Appliquer le fichier YAML de déploiement sur Kubernetes
                        kubectl apply -f k8s/deployment.yaml --namespace=zineb
                        
                        # Vérifier que le déploiement a réussi
                        kubectl rollout status deployment/frontend -n zineb --timeout=2m
                    """
                }
            }
        }
    }

    // Actions post-build (optionnel)
    post {
        success {
            echo '✅ Déploiement réussi!'
        }
        failure {
            echo '❌ Déploiement échoué!'
        }
    }
}

