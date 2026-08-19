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
        echo 'Build happens here'
      }
    }
    stage('Test'){
      steps{
        echo 'Testing happens here'
      }
    }
  }
  post{
    always{
      echo 'Pipeline has finished'
    }
  }
}
