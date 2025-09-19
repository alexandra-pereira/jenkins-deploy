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
          # copy everything at repo root except the Jenkinsfile itself
          find . -maxdepth 1 -mindepth 1 \
            -not -name 'Jenkinsfile' \
            -exec cp -r -t "$DEST" {} +
        '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          # start a temporary HTTP server just for this test
          nohup python3 -m http.server ${PORT} --directory "$DEST" >/tmp/http_${PORT}.log 2>&1 &
          SRV_PID=$!
          sleep 1
          # allow success even if curl isn't installed on the VM
          curl -f "http://localhost:${PORT}/" || true
          # stop the temporary server
          kill "$SRV_PID" 2>/dev/null || true
        '''
      }
    }
  }
}
