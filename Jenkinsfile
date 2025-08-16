// Jenkinsfile: Runs Postman tests using Newman + HTML reports inside Jenkins with Docker agent

pipeline {
  agent any  // Use any available Jenkins agent (like your local Jenkins running in Docker)

  options {
    timestamps()  // Add timestamps to console log
  }

  parameters {
    choice(
      name: 'ENV',
      choices: ['dev', 'stage'],
      description: 'Select Postman environment to run'
    )
    string(
      name: 'FOLDER',
      defaultValue: '',
      description: 'Postman folder name to run (leave empty to run all)'
    )
  }

  stages {
    stage('Install Node.js & Newman') {
      steps {
        echo '🔧 Installing Node.js & Newman dependencies...'
        sh 'node -v || true'
        sh 'npm -v || true'
        sh 'npm install -g newman newman-reporter-htmlextra || true'
      }
    }

    stage('Run API Tests with Newman') {
      steps {
        script {
          def folderArg = params.FOLDER?.trim() ? "--folder \"${params.FOLDER}\"" : ""

          sh """
            mkdir -p reports/newman
            newman run postman/AutomationExercise.postman_collection.json \\
              -e postman/environments/automationexercise-${params.ENV}.postman_environment.json \\
              ${folderArg} \\
              -r htmlextra,junit \\
              --reporter-htmlextra-export reports/newman/newman.html \\
              --reporter-junit-export    reports/newman/newman.xml \\
              --reporter-htmlextra-skipSensitiveData
          """
        }
      }
    }
  }

  post {
    always {
      echo '📦 Archiving results and publishing report...'
      junit 'reports/newman/newman.xml'

      archiveArtifacts artifacts: 'reports/newman/*', fingerprint: true

      publishHTML(target: [
        reportDir: 'reports/newman',
        reportFiles: 'newman.html',
        reportName: 'Newman HTML Report',
        alwaysLinkToLastBuild: true,
        keepAll: true
      ])
    }
  }
}
