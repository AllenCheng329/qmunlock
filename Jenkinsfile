pipeline {
  // Windows and macOS stages produce the publishable desktop installers.
  agent none

  options {
    timestamps()
    timeout(time: 90, unit: 'MINUTES')
    skipDefaultCheckout()
  }

  stages {
    stage('Build and package in parallel') {
      parallel {
        stage('Linux: Python core tests') {
          agent { label 'gha-linux' }
          steps {
            checkout scm
            sh 'python3 tests/test_decrypt.py'
          }
        }

        stage('Windows: build installer packages') {
          agent { label 'gha-windows' }
          steps {
            checkout scm
            powershell '''
              $ErrorActionPreference = 'Stop'
              $env:PYTHONUTF8 = '1'
              python tests/test_decrypt.py
              Set-Location desktop
              npm ci
              rustup target add x86_64-pc-windows-msvc
              Remove-Item -Recurse -Force src-tauri/resources/ffmpeg/macos-universal
              npm run tauri -- build --target x86_64-pc-windows-msvc
            '''
          }
          post {
            always {
              archiveArtifacts artifacts: 'desktop/src-tauri/target/x86_64-pc-windows-msvc/release/bundle/**/*.msi, desktop/src-tauri/target/x86_64-pc-windows-msvc/release/bundle/**/*.exe', allowEmptyArchive: true
            }
          }
        }

        stage('macOS: build universal installer packages') {
          agent { label 'gha-macos' }
          steps {
            checkout scm
            sh '''
              set -eux
              python3 tests/test_decrypt.py
              cd desktop
              npm ci
              rustup target add aarch64-apple-darwin x86_64-apple-darwin
              rm -rf src-tauri/resources/ffmpeg/windows-x64
              chmod 0755 src-tauri/resources/ffmpeg/macos-universal/ffmpeg
              npm run tauri -- build --bundles app --target aarch64-apple-darwin
              ./scripts/sign-and-package-macos.sh aarch64-apple-darwin
              npm run tauri -- build --bundles app --target x86_64-apple-darwin
              ./scripts/sign-and-package-macos.sh x86_64-apple-darwin
            '''
          }
          post {
            always {
              archiveArtifacts artifacts: 'desktop/src-tauri/target/aarch64-apple-darwin/release/bundle/**/*.dmg, desktop/src-tauri/target/x86_64-apple-darwin/release/bundle/**/*.dmg', allowEmptyArchive: true
            }
          }
        }
      }
    }
  }
}
