// Jenkinsfile: CI pipeline for running Postman API tests using Newman with HTML & JUnit reports

pipeline {
  // 👇 Use Docker image with Node.js pre-installed
  agent {
    docker {
      image 'node:18'            // Node.js 18 comes with npm
      args '-u root'             // Run as root to avoid permission issues inside container
    }
  }

  // 👇 Global options
  options {
    timestamps()                // Add timestamps to console output for debugging
  }

  // 👇 Define input parameters for flexibility
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
    stage('Install Dependencies') {
      steps {
        echo '🔧 Installing project dependencies (Newman, htmlextra)...'
        sh 'node -v'             // Print Node version
        sh 'npm -v'              // Print npm version
        sh 'npm install'         // Install dev dependencies listed in package.json
      }
    }

    stage('Run Newman Tests') {
      steps {
        echo "🚀 Running Newman tests for ENV: ${params.ENV} FOLDER: ${params.FOLDER}"
        script {
          // Prepare optional --folder argument
          def folderArg = params.FOLDER?.trim() ? "--folder \"${params.FOLDER}\"" : ""

          // Execute Newman CLI command
          sh """
            mkdir -p reports/newman
            npx newman run postman/AutomationExercise.postman_collection.json \\
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
      echo '📦 Archiving test results...'

      // 👇 Publish test summary in Jenkins UI
      junit 'reports/newman/newman.xml'

      // 👇 Store test artifacts (HTML report)
      archiveArtifacts artifacts: 'reports/newman/*', fingerprint: true

      // 👇 Publish detailed HTML report with link in Jenkins
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
