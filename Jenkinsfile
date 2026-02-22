pipeline {
  agent any

  tools {
    nodejs 'node-7.8.0'
    dockerTool 'docker'
  }

  environment {
        APP_PORT = ''
        DOCKER_IMAGE_NAME = ''
    }

  stages {
    stage('Prepare Environment') {
      steps {
        script {
          if (env.BRANCH_NAME == 'main') {
            env.APP_PORT = '3000'
            env.DOCKER_IMAGE_NAME = "nodemain:v1.0"
          } else if (env.BRANCH_NAME == 'dev') {
            env.APP_PORT = '3001'
            env.DOCKER_IMAGE_NAME = "nodedev:v1.0"
          }
          echo "Deploying branch ${env.BRANCH_NAME} to port ${env.APP_PORT}"
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
        echo "Deploying application..."
        sh 'docker ps -q | xargs -r docker stop'
        sh 'docker ps -aq | xargs -r docker rm'
        sh "docker run -d --expose ${env.APP_PORT} -p ${env.APP_PORT}:${env.APP_PORT} ${env.DOCKER_IMAGE_NAME}"
      }
    }
  }
}
