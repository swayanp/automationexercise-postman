pipeline {
  agent any  // Run on any available Jenkins agent

  options {
    timestamps()  // Add timestamps to Jenkins console logs
  }

  triggers {
    githubPush()  // Automatically trigger build on every GitHub push
  }

  parameters {
    // Dropdown for selecting Postman environment (e.g., dev or stage)
    choice(
      name: 'ENV',
      choices: ['dev', 'stage'],
      description: 'Select Postman environment to run'
    )

    // Optional string input to run only one specific folder in the collection
    string(
      name: 'FOLDER',
      defaultValue: '',
      description: 'Postman folder name to run (leave empty to run all)'
    )
  }

  stages {

    stage('Checkout') {
      steps {
        // Pull the GitHub repository (adjust URL/branch as needed)
        git branch: 'main', url: 'https://github.com/swayanp/automationexercise-postman.git'
      }
    }

    stage('Install Node deps') {
      steps {
        script {
          // Install Node.js dependencies differently for Unix and Windows
          if (isUnix()) {
            sh 'node -v || true'       // Check node version (skip error if not found)
            sh 'npm -v || true'        // Check npm version
            sh 'npm ci || npm install' // Prefer clean install (fall back if lock file missing)
          } else {
            bat 'node -v  || ver >NUL'
            bat 'npm -v   || ver >NUL'
            bat 'npm install'          // Use install for Windows
          }
        }
      }
    }

    stage('Run Newman') {
      steps {
        script {
          // Create folder argument dynamically only if FOLDER is passed
          def folderArg = params.FOLDER?.trim() ? "--folder \"${params.FOLDER}\"" : ""

          if (isUnix()) {
            // Run Newman on Unix environment
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
          } else {
            // Run Newman on Windows environment
            bat """
              if not exist reports\\newman mkdir reports\\newman
              npx newman run postman\\AutomationExercise.postman_collection.json ^
                -e postman\\environments\\automationexercise-${params.ENV}.postman_environment.json ^
                ${folderArg} ^
                -r htmlextra,junit ^
                --reporter-htmlextra-export reports\\newman\\newman.html ^
                --reporter-junit-export    reports\\newman\\newman.xml ^
                --reporter-htmlextra-skipSensitiveData
            """
          }
        }
      }

      post {
        always {
          // Parse and display test results (JUnit XML) in Jenkins UI
          junit 'reports/newman/newman.xml'

          // Publish fancy HTML report (Newman htmlextra) inside Jenkins
          publishHTML(target: [
            reportDir: 'reports/newman',
            reportFiles: 'newman.html',
            reportName: 'Newman HTML Report',
            keepAll: true,
            alwaysLinkToLastBuild: true,
            allowMissing: false
          ])
        }
      }
    }
  }

  post {
    always {
      // Archive all generated Newman report files so you can download from Jenkins
      archiveArtifacts artifacts: 'reports/newman/*', fingerprint: true
    }
  }
}
