pipeline {
  agent any

  environment {
    AWS_DEFAULT_REGION = 'ap-south-1'
    EB_APP_NAME        = 'hello-world-app'
    EB_ENV_NAME        = 'hello-world-env'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }

    stage('Deploy') {
      steps {
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding',
          credentialsId: 'awscredentials'
        ]]) {
          // Prepare the ZIP bundle
          sh '''
            mkdir -p deploy
            cp target/*.jar deploy/app.jar
            cd deploy
            zip -r app.zip app.jar
          '''

          // Initialize & deploy with the EB CLI
          sh '''
            eb init $EB_APP_NAME \
              --region $AWS_DEFAULT_REGION \
              --platform java \
              --keyname my-ec2-keypair
            eb deploy $EB_ENV_NAME --staged
          '''
        }
      }
    }
  }

  post {
    success {
      echo '✅ Deployment succeeded!'
    }
    failure {
      echo '❌ Deployment failed!'
    }
  }
}
