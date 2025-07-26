pipeline {
  agent any

  environment {
    AWS_DEFAULT_REGION = 'ap-south-1'
    EB_APP_NAME        = 'hello-world-app'    // your existing EB application
    EB_ENV_NAME        = 'hello-world-env'    // your existing EB environment
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
          // 1) Prepare a ZIP bundle with your JAR
          sh '''
            rm -rf deploy
            mkdir deploy
            cp target/*.jar deploy/app.jar
            cd deploy
            zip -r app.zip app.jar
          '''

          // 2) Point eb CLI at your existing app/env and deploy
          sh '''
            # Use your AWS creds + region
            export AWS_DEFAULT_REGION='${AWS_DEFAULT_REGION}'

            # Tell EB CLI which application & environment to target
            eb init ${EB_APP_NAME} --region ${AWS_DEFAULT_REGION} --no-interactive
            eb use ${EB_ENV_NAME}

            # Push the new version
            eb deploy --staged
          '''
        }
      }
    }
  }

  post {
    success {
      echo "✅ Deployed build #${env.BUILD_NUMBER} to ${EB_ENV_NAME}"
    }
    failure {
      echo "❌ Deployment of build #${env.BUILD_NUMBER} failed!"
    }
  }
}
