pipeline {
  agent any

tools { nodejs 'node18' }

  environment {
    // Jenkins injects this as env var for Snyk CLI
    SNYK_TOKEN = credentials('snyk-token')

    // Name shown in Snyk UI (helps you identify the project)
    SNYK_PROJECT_NAME = "juice-shop-jenkins"
  }

  options { timestamps() }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Verify Tools') {
      steps {
        bat '''
          echo === Tool check ===
          node --version
          npm --version
          snyk --version
        '''
      }
    }

    stage('Install Dependencies') {
      steps {
        bat '''
          echo === npm ci ===
          npm ci
        '''
      }
    }

    // 1) SCA - Upload to Snyk UI
    stage('SCA Upload to Snyk UI (monitor)') {
      steps {
        bat '''
          echo === SCA: upload to Snyk UI using snyk monitor ===
          snyk monitor --all-projects --project-name=%SNYK_PROJECT_NAME% --remote-repo-url=%GIT_URL%
        '''
      }
    }

    // 2) SAST - Upload to Snyk UI
    stage('SAST Upload to Snyk UI (code --report)') {
      steps {
        bat '''
          echo === SAST: upload to Snyk UI using snyk code test --report ===
          snyk code test --report --project-name=%SNYK_PROJECT_NAME%
        '''
      }
    }
  }

  post {
    always {
      echo "Done. Now check Snyk UI -> Projects for juice-shop-jenkins."
    }
  }
}
