pipeline{
  agent any

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
  }
  post{
    always{
      echo 'Pipeline has finished'
    }
  }
}
