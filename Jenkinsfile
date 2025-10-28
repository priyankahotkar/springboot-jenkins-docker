pipeline {
  agent any

  environment {
    // Docker repository names must be all lowercase
    IMAGE = "priyankahotkar/jenkins-dockerize"
    TAG = "latest"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        // Use 'bat' for Windows batch commands
        bat 'mvn clean package -DskipTests'
      }
    }

    stage('Build Docker Image') {
      steps {
        // Windows variable expansion uses %VARIABLE%
        bat 'docker build -t %IMAGE%:%TAG% .'
      }
    }

    stage('Run Container') {
      steps {
        // Windows batch syntax for conditional logic
        bat '''
          docker ps -a -q -f name=jenkins-demo
          if errorlevel 1 (
            echo Container not found, skipping removal.
          ) else (
            echo Container found, removing...
            docker rm -f jenkins-demo
          )
          docker run -d --name jenkins-demo -p 8080:8080 %IMAGE%:%TAG%
        '''
      }
    }

    stage('Optional: Push to Docker Hub') {
      when {
        expression { return env.DOCKER_PUSH == 'true' }
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          bat '''
            echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
            docker push %IMAGE%:%TAG%
          '''
        }
      }
    }
  }

  post {
    success {
      echo "Pipeline succeeded."
    }
    failure {
      echo "Pipeline failed."
    }
  }
}
