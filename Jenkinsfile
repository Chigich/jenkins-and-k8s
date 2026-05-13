pipeline {
  agent any

  environment {
    IMAGE_NAME = "chiragchigi/my-app"
    IMAGE_TAG  = "${BUILD_NUMBER}"           // Jenkins auto-increments this
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Chigich/jenkins-and-k8s.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
          sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
        }
      }
    }

stage('Deploy to EKS') {
  steps {
    withCredentials([
      string(credentialsId: 'aws-access-key-id',     variable: 'AWS_ACCESS_KEY_ID'),
      string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
    ]) {
      sh """
        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
        aws configure set region us-east-1


        aws eks update-kubeconfig --name cluster-1 --region us-east-1


        sed -i 's/IMAGE_TAG/${IMAGE_TAG}/g' k8s/deployment.yaml
        kubectl apply -f k8s/deployment.yaml
        kubectl rollout status deployment/my-app
      """
    }
  }
}
  post {
    success { echo "Deployed successfully — build #${BUILD_NUMBER}" }
    failure { echo "Build failed. Check logs above." }
  }
}
