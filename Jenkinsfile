pipeline {
  agent any

  tools {
    nodejs 'node-7.8.0'
  }

  stages {
    stage('Build') {
      steps {
        sh './scripts/build.sh'
      }
    }

    stage('Test') {
      steps {
        sh './scripts/test.sh'
      }
    }
  }
}
