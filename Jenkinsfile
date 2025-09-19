pipeline {
  agent any

  environment {
    DEST = "${WORKSPACE}/site"       
    BACKUP = "${WORKSPACE}/site.backup"
    PORT = "8000"                   
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
        sh 'rm -rf "$DEST" && mkdir -p "$DEST"'
        sh 'cp -r * "$DEST"/'
        sh 'rm -f "$DEST"/Jenkinsfile || true'
      }
    }

    stage('Test') {
      steps {
        sh '''
          pkill -f "http.server ${PORT}" || true
          nohup python3 -m http.server ${PORT} --directory "$DEST" >/tmp/http_${PORT}.log 2>&1 &
          sleep 1
          curl -f "http://localhost:${PORT}/" || true
        '''
      }
    }
  }

  post {
    success {
      echo 'Déploiement réussi du site web statique'
    }
    failure {
      echo 'Erreur lors du déploiement'
    }
    always {
      sh 'pkill -f "http.server ${PORT}" || true'
      sh 'rm -rf "$BACKUP" || true'
      echo 'Nettoyage effectué'
    }
  }
}
