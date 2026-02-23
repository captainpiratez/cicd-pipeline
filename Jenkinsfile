pipeline {
  agent any

  tools {
    nodejs 'node-7.8.0'
    docker 'docker'
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
        sh "docker --version"
        sh "docker version --format '{{.Client.APIVersion}}'"
        echo "Building Docker image: ${env.DOCKER_IMAGE}"
        sh "docker build -t ${env.DOCKER_IMAGE} ."
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh "docker --version"
          sh "docker version --format '{{.Client.APIVersion}}'"
          sh "docker stop ${env.APP_NAME}-${env.BRANCH_NAME} || true"
          sh "docker rm ${env.APP_NAME}-${env.BRANCH_NAME} || true"
          
          sh "docker run -d --name ${env.APP_NAME}-${env.BRANCH_NAME} -p ${env.PORT}:3000 ${env.DOCKER_IMAGE}"
        }
      }
    }
  }
}
