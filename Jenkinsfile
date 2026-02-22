pipeline {
  agent any

  tools {
    nodejs 'node-7.8.0'
    dockerTool 'docker'
  }

  environment {
    APP_NAME = "my-app"
    IMAGE_TAG = "v1.0"
    DOCKER_IMAGE = ""
    PORT = ""
  }

  stages {
    stage('Initialize') {
      steps {
        script {
          echo "Detected Branch: ${env.BRANCH_NAME}"

          if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master') {
            env.DOCKER_IMAGE = "nodemain:${env.IMAGE_TAG}"
            env.PORT = "3000"
          } else if (env.BRANCH_NAME == 'dev') {
            env.DOCKER_IMAGE = "nodedev:${env.IMAGE_TAG}"
            env.PORT = "3001"
          }
        }
      }
    }

    stage('Build') {
      steps {
        echo 'Building the application...'
        sh './scripts/build.sh'
      }
    }

    stage('Test') {
      steps {
        echo 'Testing the application...'
        sh './scripts/test.sh'
      }
    }

    stage('Docker build') {
      steps {
        echo "Building Docker image: ${env.DOCKER_IMAGE_NAME}"
        sh "docker build -t ${env.DOCKER_IMAGE_NAME} ."
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh "docker stop ${env.APP_NAME}-${env.BRANCH_NAME} || true"
          sh "docker rm ${env.APP_NAME}-${env.BRANCH_NAME} || true"
          
          sh "docker run -d --name ${env.APP_NAME}-${env.BRANCH_NAME} -p ${env.PORT}:3000 ${env.DOCKER_IMAGE}"
        }
      }
    }
  }
}
