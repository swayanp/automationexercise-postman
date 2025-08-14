// Jenkinsfile: Run Postman tests using Newman + HTML report

pipeline {
  agent any

  // 1. Use Node.js 24 you configured in Jenkins
  tools {
    nodejs 'Node24'
  }

  options {
    timestamps() // Show timestamps in console logs
  }

  stages {
    stage('Checkout') {
      steps {
        echo 'Cloning source code...'
        checkout scm
      }
    }

    stage('Install Newman & Reporters') {
      steps {
        echo 'Installing Newman and htmlextra...'
        sh 'node -v && npm -v'
        sh 'npm install -g newman newman-reporter-htmlextra'
      }
    }

    stage('Run Postman Collection') {
      steps {
        echo 'Running API tests with data-driven execution...'
        sh 'mkdir -p postman/results'
        sh '''
          newman run postman/API_Testing_Practice.postman_collection.json \
            -e postman/FakeStore_Dev.postman_environment.json \
            --iteration-data postman/product_data.csv \
            --reporters cli,htmlextra,junit \
            --reporter-htmlextra-export postman/results/results.html \
            --reporter-junit-export postman/results/results.xml
        '''
      }
    }
  }

  post {
    always {
      echo 'Archiving test results...'

      // 4. Show test trends in Jenkins
      junit 'postman/results/results.xml'

      // 5. Save HTML report as build artifact
      archiveArtifacts artifacts: 'postman/results/results.html', fingerprint: true

      // 6. Show clickable HTML report in Jenkins UI
      publishHTML(target: [
        reportDir: 'postman/results',
        reportFiles: 'results.html',
        reportName: 'Newman HTML Report',
        alwaysLinkToLastBuild: true,
        keepAll: true
      ])
    }
  }
}
