pipeline {
  agent any

  tools { nodejs 'node18' }

  environment {
    SNYK_TOKEN = credentials('snyk-token')
    SNYK_PROJECT_NAME = "juice-shop-jenkins"
    REMOTE_REPO_URL = "https://github.com/SindhuraByrapuneni/juice-shop.git"
  }

  options { timestamps() }

  stages {

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Verify Tools') {
      steps {
        bat '''
          echo === Tool check (informational only) ===

          echo Checking node:
          node --version || echo node not found

          echo Checking npm:
          npm --version || echo npm not found

          echo Checking docker:
          docker --version || echo docker not found

          echo Checking snyk:
          snyk --version
        '''
      }
    }

    stage('Generate Snyk JSON (SCA + SAST)') {
      steps {
        bat '''
          echo === SCA JSON ===
          snyk test --all-projects --json > snyk_sca.json || echo SCA scan returned non-zero

          echo === SAST JSON ===
          snyk code test --json > snyk_sast.json || echo SAST scan returned non-zero

          echo === List JSON files ===
          dir *.json || echo No JSON files found
        '''
      }
    }

    stage('Upload SCA to Snyk UI') {
      steps {
        bat '''
          echo === Upload SCA snapshot to Snyk UI ===
          snyk monitor --all-projects --remote-repo-url=%REMOTE_REPO_URL% || exit /b 0
        '''
      }
    }

    stage('Upload SAST to Snyk UI') {
      steps {
        bat '''
          echo === Upload SAST results to Snyk UI ===
          snyk code test --report || exit /b 0
        '''
      }
    }

    stage('Build Docker Image') {
      steps {
        bat '''
          echo === Building Juice Shop docker image ===
          docker build -t juice-shop:jenkins .
        '''
      }
    }

    stage('Container Scan (OS-level) - Snyk') {
      steps {
        bat '''
          echo === Snyk Container test (console) ===
          snyk container test juice-shop:jenkins --severity-threshold=low || exit /b 0

          echo === Export Container scan JSON ===
          snyk container test juice-shop:jenkins --json > snyk_container.json || exit /b 0

          echo === List JSON files ===
          dir *.json || echo No JSON files found
        '''
      }
    }

    stage('Upload Container Results to Snyk UI') {
      steps {
        bat '''
          echo === Uploading Container snapshot to Snyk UI ===
          snyk container monitor juice-shop:jenkins --project-name=juice-shop-container-jenkins || exit /b 0
        '''
      }
    }
  }

  post {
    always {
      echo "Archiving JSON reports (if present)"
      archiveArtifacts artifacts: '*.json', fingerprint: true, allowEmptyArchive: true
    }
  }
}
