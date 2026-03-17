pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  environment {
    DEPLOY_DIR = '/var/www/html'
    BUILD_DIR  = '.'             // ← працюємо з кореня репозиторію
    ARTIFACT   = 'site.tar.gz'
  }

  stages {
    stage('Checkout (SCM)') {
      steps {
        // Для "Pipeline script from SCM" checkout робиться автоматом,
        // але залишимо для надійності/кастомних агентів.
        checkout scm
      }
    }

    stage('Debug workspace') {
      steps {
        sh '''
          set -euxo pipefail
          echo "PWD: $PWD"
          ls -la
          echo "Expect to see: Jenkinsfile and index.html in repo root"
          test -f Jenkinsfile || (echo "No Jenkinsfile in PWD" && exit 2)
          test -f index.html  || (echo "No index.html in PWD" && exit 2)
        '''
      }
    }

    stage('Check Apache') {
      steps {
        sh '''
          set -euxo pipefail
          sudo systemctl is-active --quiet apache2 || sudo systemctl start apache2
          sudo systemctl enable apache2 || true
        '''
      }
    }

    stage('Prepare Artifact') {
      steps {
        sh '''
          set -euxo pipefail
          # Створюємо архів з усього вмісту репо, але виключаємо те, що не потрібно деплоїти
          tar -czf "${ARTIFACT}" \
            --exclude=".git" \
            --exclude=".gitignore" \
            --exclude="Jenkinsfile" \
            --exclude="${ARTIFACT}" \
            -C "${BUILD_DIR}" .
          echo "Created artifact: ${ARTIFACT}"
          ls -lh "${ARTIFACT}"
        '''
      }
    }

    stage('Archive artifact in Jenkins') {
      steps {
        archiveArtifacts artifacts: "${ARTIFACT}", fingerprint: true, onlyIfSuccessful: true
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          set -euxo pipefail
          # (Опційно) бекап попереднього вмісту
          if [ -d "${DEPLOY_DIR}" ]; then
            sudo mkdir -p /var/backups/www
            TS=$(date +%Y%m%d_%H%M%S)
            sudo tar -czf "/var/backups/www/html_${TS}.tar.gz" -C "${DEPLOY_DIR}" .
          fi

          # Очистка та розгортання
          sudo rm -rf "${DEPLOY_DIR:?}/"*
          sudo tar -xzf "${ARTIFACT}" -C "${DEPLOY_DIR}"

          # Права для Apache
          sudo chown -R www-data:www-data "${DEPLOY_DIR}"
          sudo find "${DEPLOY_DIR}" -type d -exec chmod 755 {} \\;
          sudo find "${DEPLOY_DIR}" -type f -exec chmod 644 {} \\;

          # Перезапуск/перечитка конфігів
          sudo systemctl reload apache2 || sudo systemctl restart apache2
        '''
      }
    }

    stage('Smoke Test') {
      steps {
        sh '''
          set -euxo pipefail
          code=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/)
          echo "HTTP status: $code"
          test "$code" -ge 200 -a "$code" -lt 400
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Deploy OK → http://localhost/"
    }
    failure {
      echo "❌ Deploy failed. Check console log and /var/log/apache2/error.log"
    }
  }
}
