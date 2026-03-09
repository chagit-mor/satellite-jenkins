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
        '''

        echo "Generating satellite transmissions..."

        bat '''
        echo ORBIT_SIGNAL > %DATA_DIR%\\orbit.txt
        echo SENSOR_SIGNAL > %DATA_DIR%\\sensor.txt
        '''

        echo "Combining satellite signals..."

        bat '''
        type %DATA_DIR%\\orbit.txt %DATA_DIR%\\sensor.txt > build\\combined_signal.txt
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

              bat '''
              echo Reading orbital signal...
              type satellite_data\\orbit.txt
              timeout /t 4
              '''

            }

          }
        }

        stage('Analyze Sensor Signals') {
          steps {

            bat '''
            echo Reading sensor signal...
            type satellite_data\\sensor.txt
            timeout /t 4
            '''

          }
        }

        stage('Validate Combined Data') {
          steps {

            bat '''
            echo Validating combined data...
            type build\\combined_signal.txt
            timeout /t 4
            '''

          }
        }

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
        '''

        junit 'results/report.xml'

      }
    }

    stage('Secure Deployment Preparation') {

      when {
        branch 'main'
      }

      steps {

        withCredentials([
          usernamePassword(
            credentialsId: 'satellite-control-access',
            usernameVariable: 'USER',
            passwordVariable: 'PASS'
          )
        ]) {

          bat '''
          echo Secure access granted for deployment
          echo Connected as %USER%
          '''

        }

        echo "Retrieving stored satellite build..."

        unstash 'satellite-build'

      }
    }

    stage('Deploy Satellite Processing System') {

      when {
        branch 'main'
      }

      steps {

        bat '''
        echo Deploying processed signal system
        type build\\combined_signal.txt
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