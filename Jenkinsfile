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
                    // Créer un réseau Docker pour garantir la communication
                    sh 'docker network create tpprojet-network || true'
                    // Vérifie si le conteneur MySQL existe
                    def mysqlRunning = sh(script: "docker ps -q -f name=tpprojet-mysql", returnStdout: true).trim()
                    if (!mysqlRunning) {
                        echo "🚀 Lancement du conteneur MySQL..."
                        sh """
                        docker run -d --name tpprojet-mysql \
                        --network tpprojet-network \
                        -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} \
                        -e MYSQL_DATABASE=${MYSQL_DB} \
                        -e MYSQL_USER=${MYSQL_USER} \
                        -e MYSQL_PASSWORD=${MYSQL_PASSWORD} \
                        -p 3306:3306 \
                        mysql:8
                        """
                    } else {
                        echo "✅ Conteneur MySQL déjà en cours d'exécution."
                        sh "docker start tpprojet-mysql || true"
                    }
                    // Attendre que MySQL soit prêt
                    sh '''
                    echo "⏳ Attente que MySQL soit prêt..."
                    for i in {1..10}; do
                        if docker exec tpprojet-mysql mysql -u${MYSQL_USER} -p${MYSQL_PASSWORD} -e "SELECT 1" ${MYSQL_DB} >/dev/null 2>&1; then
                            echo "✅ MySQL est prêt !"
                            exit 0
                        fi
                        echo "MySQL non prêt, nouvelle tentative dans 5 secondes..."
                        sleep 5
                    done
                    echo "❌ MySQL ne répond pas après 50 secondes."
                    exit 1
                    '''
                    // Configurer les permissions MySQL pour root@'%'
                    sh '''
                    docker exec tpprojet-mysql mysql -u${MYSQL_ROOT_PASSWORD} -p${MYSQL_ROOT_PASSWORD} -e \
                    "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '${MYSQL_ROOT_PASSWORD}'; FLUSH PRIVILEGES;"
                    '''
                }
            }
        }
        // ... (autres étapes inchangées : Pull from Git, Clean, Compile, SonarQube Analysis)
        // 5️⃣ Génération du JAR avec profil "test"
        stage('Build JAR') {
            steps {
                script {
                    // Override des propriétés de connexion
                    withEnv([
                        "SPRING_DATASOURCE_URL=jdbc:mysql://tpprojet-mysql:3306/${MYSQL_DB}?createDatabaseIfNotExist=true",
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
    post {
        success {
            echo '✅ Pipeline terminé avec succès !'
        }
        failure {
            echo '❌ Erreur dans le pipeline, vérifier les logs.'
        }
    }
}
