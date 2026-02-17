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
      echo === Tool check (informational only) ===

      echo Checking node:
      node --version || echo node not found

      echo Checking npm:
      npm --version || echo npm not found

      echo Checking snyk:
      snyk --version
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
    '''
  }
}

stage('Upload Container Results to Snyk UI (monitor)') {
  steps {
    bat '''
      echo === Uploading Container snapshot to Snyk UI ===
      snyk container monitor juice-shop:jenkins --project-name=juice-shop-container-jenkins || exit /b 0
    '''
  }
}



    
    // 1) SCA - Upload to Snyk UI
    stage('SCA Upload to Snyk UI (monitor)') {
      steps {
        bat '''
          echo === SCA: upload to Snyk UI using snyk monitor ===
          snyk monitor --file=package.json --project-name=juice-shop-jenkins --remote-repo-url=https://github.com/SindhuraByrapuneni/juice-shop.git || exit /b 0

        '''
      }
    }

    // 2) SAST - Upload to Snyk UI
    stage('SAST Upload to Snyk UI (code --report)') {
      steps {
        bat '''
          echo === Uploading SAST results to Snyk UI ===
      snyk code test --report || exit /b 0
     
        '''
      }
    }
  }

post {
    always {
      echo 'Archiving Snyk security reports'
      archiveArtifacts artifacts: 'snyk_sca.json,snyk_sast.json,snyk_container.json', fingerprint: true
    }
  }

  }
