pipeline {
    agent any

    environment {
        
        SONARQUBE_SERVER = 'SonarQube'
    }

    stages {
        // 1️⃣ Pull depuis Git
        stage('Pull from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/lakhalemna/TPFoyer-DevOps.git'
            }
        }

        // 2️⃣ Nettoyage du projet
        stage('Clean') {
            steps {
                sh 'mvn clean'
            }
        }

        // 3️⃣ Compilation
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        // 4️⃣ Analyse de la qualité du code avec SonarQube
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=TPFoyer -Dsonar.host.url=http://192.168.33.10:9000 -Dsonar.login=squ_8eb92e72c4010669e9fe8e78b82a771f4c2975f5'
                }
            }
        }

        // 5️⃣ Génération du JAR
        stage('Build JAR') {
            steps {
                sh 'mvn package'
            }
        }
    }

    post {
        success {
            echo 'Pipeline terminé avec succès !'
        }
        failure {
            echo 'Erreur dans le pipeline, vérifier les logs.'
        }
    }
}
