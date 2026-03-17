pipeline {
    agent any

    environment {
        DEPLOY_DIR = '/var/www/html'
        BUILD_DIR  = 'site'
        ARTIFACT   = 'site.tar.gz'
    }

    stages {
        stage('Validate Apache status') {
            steps {
                sh '''
                  sudo systemctl is-active --quiet apache2 || sudo systemctl start apache2
                  sudo systemctl enable apache2 || true
                '''
            }
        }

        stage('Prepare artifact') {
            steps {
                sh '''
                  tar -czf "${ARTIFACT}" -C "${BUILD_DIR}" .
                '''
            }
        }

        stage('Deploy to Apache docroot') {
            steps {
                sh '''
                  sudo rm -rf "${DEPLOY_DIR:?}/"*
                  sudo mkdir -p "${DEPLOY_DIR}"
                  sudo tar -xzf "${ARTIFACT}" -C "${DEPLOY_DIR}"
                  sudo chown -R www-data:www-data "${DEPLOY_DIR}"
                '''
            }
        }

        stage('Smoke test') {
            steps {
                sh '''
                  curl -I http://localhost/
                '''
            }
        }
    }
}
