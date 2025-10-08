pipeline {
    agent any
    environment {
        SONARQUBE_SERVER = 'SonarQube'
        MYSQL_DB = 'tpprojet'
        MYSQL_ROOT_PASSWORD = 'root'
        MYSQL_HOST = '192.168.33.10' // Remplacez par l'IP de votre machine Vagrant si nécessaire
    }
    stages {
        // 1️⃣ Pull depuis Git
        stage('Pull from Git') {
            steps {
                // Pour un dépôt public, pas besoin de credentialsId
                // Si privé, ajoutez credentialsId: 'github-credentials'
                git branch: 'main', url: 'https://github.com/lakhalemna/TPFoyer-DevOps.git'
            }
        }
        // 2️⃣ Nettoyage du projet
        stage('Clean') {
            steps {
                sh 'mvn clean'
            }
        }
        // 3️⃣ Compilation du projet
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
       // 4️⃣ Analyse de la qualité du code avec SonarQube
        stage('SonarQube Analysis'){
            steps {
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=TPFoyer -Dsonar.host.url=http://192.168.33.10:9000 -Dsonar.login=squ_8eb92e72c4010669e9fe8e78b82a771f4c2975f5' 
                }
            } 
        }
        // 5️⃣ Génération du fichier JAR
        stage('Build JAR') {
            steps {
                script {
                    // Override des propriétés de connexion
                    withEnv([
                        "SPRING_DATASOURCE_URL=jdbc:mysql://${MYSQL_HOST}:3306/${MYSQL_DB}?createDatabaseIfNotExist=true",
                        "SPRING_DATASOURCE_USERNAME=root",
                        "SPRING_DATASOURCE_PASSWORD=${MYSQL_ROOT_PASSWORD}",
                        "SPRING_JPA_PROPERTIES_HIBERNATE_DIALECT=org.hibernate.dialect.MySQLDialect"
                    ]) {
                        sh 'mvn package -Dspring.profiles.active=test'
                    }
                }
            }
        }
    }
}
