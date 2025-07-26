pipeline {
  agent any
  environment {
    AWS_ACCESS_KEY_ID     = credentials('awscredentials').accessKey
    AWS_SECRET_ACCESS_KEY = credentials('awscredentials').secretKey
    AWS_DEFAULT_REGION    = 'ap-south-1'
    EB_APP_NAME           = 'hello-world-app'
    EB_ENV_NAME           = 'hello-world-env'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build') {
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }
    stage('Deploy') {
      steps {
        // zip the Spring Boot jar
        sh '''
          mkdir -p deploy
          cp target/*.jar deploy/app.jar
          cd deploy
          zip -r app.zip app.jar
        '''
        // deploy via EB CLI
        sh '''
          eb init $EB_APP_NAME --region $AWS_DEFAULT_REGION
          eb deploy $EB_ENV_NAME --staged
        '''
      }
    }
  }
}
