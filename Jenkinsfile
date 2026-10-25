pipeline {
  agent any
  triggers { pollSCM('H/2 * * * *') }
  environment {
    AWS_ENDPOINT_URL      = 'http://host.docker.internal:4566'
    AWS_DEFAULT_REGION    = 'us-east-1'
    AWS_ACCESS_KEY_ID     = 'test'
    AWS_SECRET_ACCESS_KEY = 'test'
    PUSH_REG = 'localhost:5100'
    PULL_REG = 'host.minikube.internal:5100'
    K8S_SERVER = 'https://192.168.58.2:8443'
  }
  stages {
    stage('Build') {
      steps { sh 'docker build -t $PUSH_REG/hello-app:$BUILD_NUMBER .' }
    }
    stage('Push to ECR') {
      steps {
        sh '''
          aws ecr get-login-password | docker login --username AWS --password-stdin $PUSH_REG
          docker push $PUSH_REG/hello-app:$BUILD_NUMBER
        '''
      }
    }
    stage('Deploy') {
      steps {
        withCredentials([string(credentialsId: 'k8s-token', variable: 'TOKEN')]) {
          sh '''
            sed "s|IMAGE_PLACEHOLDER|$PULL_REG/hello-app:$BUILD_NUMBER|" k8s/deployment.yaml > /tmp/deploy.yaml
            kubectl --server=$K8S_SERVER --token=$TOKEN --insecure-skip-tls-verify apply -f /tmp/deploy.yaml
            kubectl --server=$K8S_SERVER --token=$TOKEN --insecure-skip-tls-verify rollout status deployment/hello-app --timeout=120s
          '''
        }
      }
    }
  }
}
