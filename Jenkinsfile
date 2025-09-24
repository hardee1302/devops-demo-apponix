pipeline {
  agent any

  environment {
    IMAGE_NAME = "devops-demo"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build (Maven)') {
      steps {
        sh 'mvn -B -DskipTests clean package'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          def tag = "${env.BUILD_NUMBER}"
          sh "eval $(minikube -p minikube docker-env) && docker build -t ${IMAGE_NAME}:${tag} ."
          sh "eval $(minikube -p minikube docker-env) && docker tag ${IMAGE_NAME}:${tag} ${IMAGE_NAME}:latest"
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh '''
        eval $(minikube -p minikube docker-env)
        kubectl set image deployment/devops-demo devops-demo=${IMAGE_NAME}:${BUILD_NUMBER} --record || kubectl apply -f k8s/
        kubectl rollout status deployment/devops-demo --timeout=120s
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline succeeded!"
    }
    failure {
      echo "❌ Pipeline failed! try again"
    }
  }
}
