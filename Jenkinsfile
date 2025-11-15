pipeline {
  agent any

  environment {
    SONARQUBE_SERVER = 'SonarQube'
    SCAN_TARGET_URL = 'http://host.docker.internal:3000'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('SAST - SonarQube') {
      steps {
        withSonarQubeEnv('SonarQube') {
          bat 'sonar-scanner'
        }
      }
    }

    stage('SCA - Dependency-Check') {
      steps {
        bat '''
          dependency-check.bat ^
            --project JuiceShop ^
            --scan . ^
            --format HTML ^
            --out reports\\dependency-check
        '''
        archiveArtifacts artifacts: 'reports/dependency-check/dependency-check-report.html', fingerprint: true
      }
    }

    stage('Deploy test env') {
      steps {
        bat 'docker compose -f docker-compose.test.yml up -d'
      }
    }

    stage('DAST - ZAP Baseline') {
      steps {
        bat """
          docker run --rm -p 9090:9090 ghcr.io/zaproxy/zaproxy:stable ^
            zap-baseline.py -t ${SCAN_TARGET_URL} -r zap-baseline.html
        """
        archiveArtifacts artifacts: 'zap-baseline.html', fingerprint: true
      }
    }
  }

  post {
    always {
      bat 'docker compose -f docker-compose.test.yml down || exit 0'
    }
  }
}
