pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQube'
        MYSQL_USER = 'user'
        MYSQL_PASSWORD = 'password'
        MYSQL_DB = 'tpprojet'
        MYSQL_ROOT_PASSWORD = 'root'
    }

    stages {
        // 0️⃣ Lancer MySQL dans Docker
        stage('Start MySQL') {
            steps {
                script {
                    // Vérifie si le conteneur MySQL existe
                    def mysqlRunning = sh(script: "docker ps -q -f name=tpprojet-mysql", returnStdout: true).trim()
                    if (!mysqlRunning) {
                        // Crée et lance le conteneur
                        sh """
                        docker run -d --name tpprojet-mysql \
                        -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} \
                        -e MYSQL_DATABASE=${MYSQL_DB} \
                        -e MYSQL_USER=${MYSQL_USER} \
                        -e MYSQL_PASSWORD=${MYSQL_PASSWORD} \
                        -p 3306:3306 \
                        mysql:8
                        """
                    } else {
                        // Démarre le conteneur s'il est arrêté
                        sh "docker start tpprojet-mysql"
                    }

                    // Attendre que MySQL soit prêt
                    sh "sleep 20"
                }
            }
        }

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
