pipeline {
  agent any

  environment {
    DEST   = "${WORKSPACE}/site"
    BACKUP = "${WORKSPACE}/site.backup"
    PORT   = "8000"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Backup') {
      steps {
        sh 'test -d "$DEST" && cp -r "$DEST" "$BACKUP" || true'
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          rm -rf "$DEST" && mkdir -p "$DEST"
          find . -maxdepth 1 -mindepth 1 \
            -not -name 'Jenkinsfile' \
            -not -name 'site' \
            -not -name 'site.backup' \
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
      sh 'rm -rf "$BACKUP" || true'
      echo 'Nettoyage effectué'
    }
  }
}
