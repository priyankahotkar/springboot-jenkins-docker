pipeline {
  agent any

  environment {
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
        sh 'mvn clean package -DskipTests'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t ${IMAGE}:${TAG} .'
      }
    }

    stage('Run Container') {
      steps {
        sh '''
          if [ "$(docker ps -q -f name=jenkins-demo)" ]; then
            docker rm -f jenkins-demo || true
          fi
          docker run -d --name jenkins-demo -p 8080:8080 ${IMAGE}:${TAG}
        '''
      }
    }

    stage('Optional: Push to Docker Hub') {
      when {
        expression { return env.DOCKER_PUSH == 'true' }
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${IMAGE}:${TAG}
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
