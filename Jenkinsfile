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
                        echo "🚀 Lancement du conteneur MySQL..."
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
                        echo "✅ Conteneur MySQL déjà en cours d'exécution."
                        sh "docker start tpprojet-mysql || true"
                    }

                    // ✅ Attendre que MySQL accepte les connexions
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
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=TPFoyer \
                    -Dsonar.host.url=http://192.168.33.10:9000 \
                    -Dsonar.login=squ_8eb92e72c4010669e9fe8e78b82a771f4c2975f5
                    '''
                }
            }
        }

        // 5️⃣ Génération du JAR avec profil "test"
        stage('Build JAR') {
            steps {
                // ✅ Correction ici : on remplace -Ptest par -Dspring.profiles.active=test
                sh 'mvn package -Dspring.profiles.active=test'
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
