pipeline {
    agent any

    stages {
        stage('Start Apache') {
            steps {
                sh '''
                  sudo systemctl start apache2 || true
                  sudo systemctl enable apache2 || true
                  echo "Apach2 is STARTED by Andryy from Git via Jenkins"
                '''
            }
        }

        stage('Check Apache Status') {
            steps {
                sh '''
                  echo "Status Apache:"
                  sudo systemctl status apache2 | grep Active
                '''
            }
        }

        stage('Show Test Output') {
            steps {
                sh '''
                  echo "--------------------------------"
                  echo "     Apach2 is STARTED by Andryy from Git via Jenkins!"
                  echo "--------------------------------"
                '''
            }
        }
    }

    post {
        success {
            echo "✔ Пайплайн успішно виконано!"
        }
        failure {
            echo "❌ Помилка у пайплайні!"
        }
    }
}
