pipeline {

  agent any

  environment {
    APP_ENV = "staging"
    DATA_DIR = "satellite_data"
  }

  options {
    timeout(time: 20, unit: 'MINUTES')
    timestamps()
  }

  stages {

    stage('Receive Satellite Mission Code') {
      steps {
        echo "Receiving satellite mission code..."
        checkout scm
      }
    }

    stage('Build Satellite Processing Engine') {
      steps {
        echo "Preparing satellite processing directories..."
        bat '''
        if not exist build mkdir build
        if not exist %DATA_DIR% mkdir %DATA_DIR%
        dir
        '''

        echo "Generating satellite transmissions..."
        bat '''
        echo ORBIT_SIGNAL > %DATA_DIR%\\orbit.txt
        echo SENSOR_SIGNAL > %DATA_DIR%\\sensor.txt
        dir %DATA_DIR%
        '''

        echo "Combining satellite signals..."
        bat '''
        type %DATA_DIR%\\orbit.txt %DATA_DIR%\\sensor.txt > build\\combined_signal.txt
        dir build
        '''

        echo "Storing build artifacts for future stages..."
        stash includes: 'build/**', name: 'satellite-build'
      }
    }

    stage('Satellite Signal Analysis') {

      parallel {

        stage('Analyze Orbital Signals') {
          steps {

            retry(2) {
              echo "Analyzing orbital signals (retry demo)..."
              bat '''
              if not exist fail.marker (
              echo First attempt failed > fail.marker
              exit /b 1
              )
              echo Step passed
              '''
              sleep time: 4, unit: 'SECONDS'
            }

          }
        }

        stage('Analyze Sensor Signals') {
          steps {
            echo "Analyzing sensor signals..."
            bat '''
            type %DATA_DIR%\\sensor.txt
            '''
            sleep time: 4, unit: 'SECONDS'
          }
        }

        stage('Validate Combined Data') {
          steps {
            echo "Validating combined data..."
            bat '''
            type build\\combined_signal.txt
            '''
            sleep time: 4, unit: 'SECONDS'
          }
        }

      }

    }
    stage('Print Branch Name') {
        steps {
            echo "The current branch is: ${env.BRANCH_NAME}"
            echo "The current branch is: ${env.GIT_BRANCH}"
        }
    }

    stage('Generate Test Report') {
      steps {

        bat '''
        if not exist results mkdir results
        echo ^<testsuite name="satellite-tests"^> > results\\report.xml
        echo ^<testcase classname="orbit" name="orbitTest"/^> >> results\\report.xml
        echo ^<testcase classname="sensor" name="sensorTest"/^> >> results\\report.xml
        echo ^</testsuite^> >> results\\report.xml
        dir results
        type results\\report.xml
        '''

        junit 'results/report.xml'

      }
    }

    stage('Secure Deployment Preparation') {

      when {
        expression { env.GIT_BRANCH == 'origin/main' }
      }

      steps {

        withCredentials([
          usernamePassword(
            credentialsId: 'satellite-control-access',
            usernameVariable: 'USER',
            passwordVariable: 'PASS'
          )
        ]) {

          echo "Secure access granted for deployment"
          bat '''
          echo Connected as %USER%
          '''

        }

        echo "Retrieving stored satellite build..."
        unstash 'satellite-build'
        bat 'dir build'
      }
    }

    stage('Deploy Satellite Processing System') {
      steps {
        echo "Deploying processed signal system..."
        bat '''
        type build\\combined_signal.txt
        dir build
        '''

        archiveArtifacts artifacts: 'build/**'
      }

    }

  }

  post {

    success {
      echo "Satellite processing mission completed successfully."
    }

    failure {
      echo "Satellite mission encountered an error."
    }

    always {
      echo "Cleaning workspace after mission..."
      deleteDir()
    }

  }

}