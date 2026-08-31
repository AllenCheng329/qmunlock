pipeline {
  agent { label 'gha-linux' }

  options {
    timestamps()
    timeout(time: 20, unit: 'MINUTES')
  }

  stages {
    stage('Python core tests') {
      steps {
        sh 'python3 tests/test_decrypt.py'
      }
    }
    stage('Desktop frontend build') {
      steps {
        dir('desktop') {
          sh 'npm ci'
          sh 'npm run build'
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'desktop/dist/**', allowEmptyArchive: true
    }
  }
}