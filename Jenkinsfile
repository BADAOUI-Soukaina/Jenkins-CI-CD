pipeline {
    agent any

    environment {
        // --- Variables d'Environnement ---
        GITHUB_TOKEN = credentials('github-token') // ID de l'Identifiant Jenkins
        VENV_DIR = ".venv"
        HOST = "0.0.0.0"
        PORT = "5000"
        APP_MODULE = "app:app"
        
        // --- Adresses E-mail Corrigées ---
        EMAIL_TO = "soukabadaoui88@gmail.com" // Votre adresse pour recevoir les notifications
        // L'adresse FROM est l'adresse Gmail configurée dans l'admin Jenkins
        EMAIL_FROM = "soukabadaoui88@gmail.com" 
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning code from GitHub...'
                // Utilise votre URL de dépôt corrigée
                git branch: 'main',
                    url: 'https://github.com/BADAOUI-Soukaina/Jenkins-CI-CD.git',
                    credentialsId: 'github-token' // Utilisé si le dépôt devient privé
            }
        }

        stage('Setup Virtual Environment') {
            steps {
                echo 'Setting up Python Virtual Environment (Windows commands)'
                // Utilisation de 'bat' pour les commandes Windows
                bat """
                // Crée le venv
                python -m venv ${VENV_DIR}
                // Active le venv
                call ${VENV_DIR}\\Scripts\\activate
                // Mise à jour de pip et installation des dépendances
                python -m pip install --upgrade pip
                pip install -r requirements.txt
                """
            }
        }

        stage('Run Tests') {
            // Exécution Conditionnelle : n'exécute pas les tests si seul README.md est modifié
            when {
                not {
                    changeset "README.md"
                }
            }
            
            // Parallélisation des tests
            parallel {
                stage('Test File 1') {
                    steps {
                        bat """
                        call ${VENV_DIR}\\Scripts\\activate
                        python -m pytest test_app.py -v
                        """
                    }
                }
                stage('Test File 2') {
                    steps {
                        bat """
                        // Assurez-vous d'avoir ce fichier dans votre dépôt !
                        call ${VENV_DIR}\\Scripts\\activate
                        python -m pytest test_app_2.py -v
                        """
                    }
                }
            }
        }

        stage('Deploy (Local with Gunicorn)') {
            steps {
                echo 'Starting Flask app locally using Gunicorn...'
                // Déploiement en arrière-plan avec 'start /B' pour ne pas bloquer Jenkins
                bat """
                call ${VENV_DIR}\\Scripts\\activate
                start /B gunicorn --bind ${HOST}:${PORT} ${APP_MODULE} > gunicorn.log 2>&1
                echo ✅ Gunicorn started on http://${HOST}:${PORT}
                """
            }
        }
    }

    // --- Notifications Post-Build ---
    post {
        success {
            emailext(
                to: env.EMAIL_TO,
                from: env.EMAIL_FROM,
                replyTo: env.EMAIL_FROM,
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                mimeType: 'text/html',
                body: """\
                <p>Good news! All stages passed.</p>
                <p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> succeeded.</p>
                <p>Check details: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """
            )
        }

        failure {
            emailext(
                to: env.EMAIL_TO,
                from: env.EMAIL_FROM,
                replyTo: env.EMAIL_FROM,
                subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                mimeType: 'text/html',
                body: """\
                <p>Uh oh...</p>
                <p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> failed.</p>
                <p>Check logs: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """
            )
        }
    }
}
