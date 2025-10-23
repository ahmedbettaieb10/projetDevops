pipeline {
    agent any

    environment {
        SONARQUBE = 'SonarQube-Server'
        DOCKERHUB_CREDENTIALS = 'dockerhub-cred'
        IMAGE_NAME = "ahmed535/springboot-app"
        KUBE_CONFIG = "$HOME/.kube/config"
    }

    stages {
        stage('Pull from Git') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/ahmedbettaieb10/projetDevops.git',
                    credentialsId: 'git-credentials'
            }
        }

        stage('Clean Project') {
            steps {
                sh 'mvn clean'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=springboot-app -Dsonar.host.url=http://localhost:9000 -Dsonar.login=$SONAR_AUTH_TOKEN'
                }
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKERHUB_CREDENTIALS}") {
                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()
                    }
                }
            }
        }

        // --- Nouvelle partie CD ---
        stage('Deploy MySQL and Backend on Minikube') {
            steps {
                script {
                    // Démarrer Minikube s’il n’est pas actif
                    sh 'minikube status || minikube start'

                    // Appliquer les fichiers YAML
                    sh '''
                        kubectl apply -f mysql-secret.yaml
                        kubectl apply -f mysql_deployment.yaml
                        kubectl apply -f restaurant-app-deployment.yaml
                    '''

                    // Vérifier les pods
                    sh 'kubectl get pods'
                }
            }
        }
    }

    post {
        success {
            echo "✅ Déploiement réussi sur Minikube !"
            sh 'kubectl get svc'
        }
        failure {
            echo "❌ Le pipeline a échoué."
        }
    }
}
