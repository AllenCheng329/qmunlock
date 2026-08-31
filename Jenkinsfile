pipeline {
  // This project doubles as a hosted-agent integration check.
  agent none

  options {
    timestamps()
    timeout(time: 30, unit: 'MINUTES')
    skipDefaultCheckout()
  }

  stages {
    stage('Linux: tests and frontend build') {
      agent { label 'gha-linux' }
      steps {
        checkout scm
        sh 'python3 tests/test_decrypt.py'
        dir('desktop') {
          sh 'npm ci'
          sh 'npm run build'
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'desktop/dist/**', allowEmptyArchive: true
        }
      }
    }

    stage('Windows: Python core tests') {
      agent { label 'gha-windows' }
      steps {
        checkout scm
        bat 'set PYTHONUTF8=1 && python tests\\test_decrypt.py'
      }
    }

    stage('macOS: Python core tests') {
      agent { label 'gha-macos' }
      steps {
        checkout scm
        sh 'python3 tests/test_decrypt.py'
      }
    }
  }
}
