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

        
        stage('Deploy index.html') {
            steps {
                sh '''
                  # Копіюємо index.html з кореня репозиторію у веб-каталог
                  sudo cp index.html /var/www/html/index.html
                  sudo chown www-data:www-data /var/www/html/index.html
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
