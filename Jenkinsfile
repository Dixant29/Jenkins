pipeline{
  agent any
  environment{
    APP_NAME = 'jenkins-learning'
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
    stage('Archive artifact'){
      steps{
        archiveArtifacts artifacts:'output/built-info.txt',fingerprint: true
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
  }
  post{
    always{
      echo 'Pipeline has finished'
    }
  }
}
