pipeline {
    agent { label 'worker-node' }

    environment {
        APP_NAME = "django-to-do-app"
        ARTIFACT = "django-to-do-app.zip"
    }

    stages {

        stage('Git Checkout') {
            steps {
                echo "Cloning GitHub Repository..."

                git branch: 'main',
                    url: 'https://github.com/Luko575/django-to-do-app.git'
            }
        }

        stage('Build Python Application') {
            steps {
                sh '''
                echo "Creating Python Virtual Environment..."

                python3 -m venv venv

                . venv/bin/activate

                pip install --upgrade pip

                pip install -r requirements.txt

                python manage.py collectstatic --noinput || true

                python manage.py migrate || true

                python manage.py test || true
                '''
            }
        }

        stage('Package & Upload to JFrog') {
            steps {
                sh '''
                echo "Packaging Application..."

                zip -r ${ARTIFACT} . -x "venv/*" ".git/*"

                echo "Uploading Artifact to JFrog..."

                jf rt upload ${ARTIFACT} python-local/
                '''
            }
        }

	 stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=django-to-do-app \
                        -Dsonar.projectName=django-to-do-app \
                        -Dsonar.sources=. \
                        -Dsonar.python.version=3
                    '''
                }
            }
        }
        
        stage('Deploy Application') {
            steps {
                sh '''
                echo "Stopping Existing Application..."

                pkill -f "python manage.py runserver" || true

                . venv/bin/activate

                echo "Starting Django Application..."

                nohup python manage.py runserver 0.0.0.0:8000 > app.log 2>&1 &
                '''
            }
        }

    }

    post {

        success {
            echo "===================================="
            echo "Pipeline completed successfully."
            echo "Application deployed on port 8000."
            echo "===================================="
        }

        failure {
            echo "Pipeline Failed."
        }

        always {
            cleanWs()
        }
    }
}
