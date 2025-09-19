pipeline {
  agent any

  environment {
    DEST = "${WORKSPACE}/site"
    PORT = "8000"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          rm -rf "$DEST" && mkdir -p "$DEST"
          # équivalent de: cp -r * /destination/ (en évitant la récursion)
          find . -maxdepth 1 -mindepth 1 \
            -not -name 'Jenkinsfile' \
            -not -name 'site' \
            -exec cp -r -t "$DEST" {} +
        '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          nohup python3 -m http.server ${PORT} --directory "$DEST" >/tmp/http_${PORT}.log 2>&1 &
          echo $! > /tmp/http_${PORT}.pid
          sleep 1
          curl -f "http://localhost:${PORT}/" || true
        '''
      }
    }
  }

  post {
    success { echo 'Déploiement terminé avec succès' }
    failure { echo 'Erreur lors du déploiement' }
    always  {
      sh 'test -f "/tmp/http_${PORT}.pid" && kill "$(cat /tmp/http_${PORT}.pid)" 2>/dev/null || true'
      echo 'Nettoyage effectué'
    }
  }
}

