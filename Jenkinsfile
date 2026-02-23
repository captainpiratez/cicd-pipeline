pipeline {
  agent any

  tools {
    nodejs 'node-7.8.0'
    dockerTool 'docker'
  }

  environment {
    APP_NAME = "my-app"
    IMAGE_TAG = "v1.0"
    REGISTRY_NAMESPACE = "captainpiratez"
  }

  stages {
    stage('Initialize') {
      steps {
        script {
          echo "Detected Branch: ${env.BRANCH_NAME}"

          if (env.BRANCH_NAME == 'main') {
            env.DOCKER_IMAGE = "nodemain:${env.IMAGE_TAG}"
            env.PORT = "3000"
          } else if (env.BRANCH_NAME == 'dev') {
            env.DOCKER_IMAGE = "nodedev:${env.IMAGE_TAG}"
            env.PORT = "3001"
          } else {
            // Default for other branches
            env.DOCKER_IMAGE = "${env.APP_NAME}:${env.IMAGE_TAG}"
            env.PORT = "3000"
          }
          
          echo "Docker Image: ${env.DOCKER_IMAGE}"
          echo "Port: ${env.PORT}"
        }
      }
    }

    stage('Hadolint') {
      steps {
        sh 'hadolint Dockerfile'
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
        echo "Building Docker image: ${env.REGISTRY_NAMESPACE}/${env.DOCKER_IMAGE}"
        sh "docker build -t ${env.REGISTRY_NAMESPACE}/${env.DOCKER_IMAGE} ."
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh "docker login -u ${env.DOCKER_USER} -p ${env.DOCKER_PASS}"
          sh "docker push ${env.REGISTRY_NAMESPACE}/${env.DOCKER_IMAGE}"
        }
      }
    }

    stage('Scan Docker Image for Vulnerabilities') {
      steps {
        script {
          def vulnerabilities = sh(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${env.REGISTRY_NAMESPACE}/${env.DOCKER_IMAGE}", returnStdout: true).trim()
          echo "Vulnerabilities Report:\n${vulnerabilities}"
        }
      }
    }
  }

  post {
    success {
      script {
        if (env.BRANCH_NAME == 'main') {
          build job: 'Deploy_to_main'
        } else if (env.BRANCH_NAME == 'dev') {
          build job: 'Deploy_to_dev'
        }
      }
    }
  }
}
