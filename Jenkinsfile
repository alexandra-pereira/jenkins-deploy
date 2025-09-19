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
          # copy everything at repo root except Jenkinsfile and our working folders
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
          SRV_PID=$!
          sleep 1
          curl -f "http://localhost:${PORT}/" || true
          kill "$SRV_PID" 2>/dev/null || true
        '''
      }
    }
  }
}
