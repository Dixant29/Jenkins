pipeline{
  agent any
  environment{
    APP_NAME = 'jenkins-learning'
  }
  parameters{
    choice(
      name: 'TARGET_ENV',
      choices: ['development','staging'],
      description: 'Where this build is intended to run'
    )
  }

  stages{
    stage('Hello'){
     steps{
       echo 'Hello to jenkins'
     }
    }
    stage('Build'){
      steps{
        sh '''
          echo "Building from: $(pwd)"
          ls -la
          mkdir -p output
          echo "Build completed" > output/built-info.txt
        '''
      }
    }
    stage('Test'){
      steps{
        sh '''
          test -f output/built-info.txt
          echo 'Test passed'
        '''
      }
    }
    stage('Show build informtion'){
      steps{
        sh '''
          echo "Application: $APP_NAME"
          echo "Jenkins build number: $BUILD_NUMBER"
          echo "Application: $APP_NAME" > output/build-metadata.txt
          echo "Build number: $BUILD_NUMBER" >> output/build-metadata.txt
        '''
      }
    }
    stage('Show selected environment'){
      steps{
        echo "Selected environment: ${params.TARGET_ENV}"

        sh '''
          echo "Target environment: ${params.TARGET_ENV}" >> output/build-metadata.txt
        '''
      }
    }
    stage('Archive artifact'){
      steps{
        archiveArtifacts artifacts:'output/*',fingerprint: true
      }
    }
  }
  post{
    always{
      echo 'Pipeline has finished'
    }
  }
}
