pipeline {
    agent any

    tools {
        maven 'Maven-3.8.8'   // nom configuré dans Jenkins
        jdk 'jdk17'           // ajoute Java 17 dans Jenkins si besoin
    }

    environment {
        SONARQUBE = 'SonarQube'   // nom configuré dans "Manage Jenkins"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/ahmedbettaieb10/projetDevops.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=TP-Projet \
                          -Dsonar.projectName=TP-Projet \
                          -Dsonar.host.url=http://192.168.33.10:9000
                    '''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
