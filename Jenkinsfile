pipeline {
    agent any

    environment {
        // Nom du serveur Sonar défini dans Jenkins
        SONARQUBE = 'SonarQube-Server'
        // Nom du repo dockerhub défini dans Jenkins
        DOCKERHUB_CREDENTIALS = 'dockerhub-cred'
        IMAGE_NAME = "ahmed535/springboot-app"
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
                        script {
                            sh """
                                docker build -t ahmed535/springboot-app:${BUILD_NUMBER} .
                            """
                        }
                    }
                }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKERHUB_CREDENTIALS}") {
                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()
                    }
                }
            }
        }

        stage('Deploy Container') {
            steps {
                script {
                    // Stoppe le conteneur s'il existe
                    sh 'docker ps -q --filter "name=springboot-app" | grep -q . && docker stop springboot-app || true'

                    // Supprime le conteneur s'il existe
                    sh 'docker ps -a -q --filter "name=springboot-app" | grep -q . && docker rm springboot-app || true'

                    // Déploie le nouveau conteneur avec le build en cours
                    sh "docker run -d --name springboot-app -p 9090:8080 ahmed535/springboot-app:${env.BUILD_NUMBER}"
                }
            }
        }
    }
}
