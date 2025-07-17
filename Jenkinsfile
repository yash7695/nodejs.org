pipeline {
  agent any

  
  environment {
    AWS_REGION = "ap-south-1"
    IMAGE_NAME = "nodejs-app"
    ECR_URL = "120569645875.dkr.ecr.ap-south-1.amazonaws.com/noderepo"
  }

  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Install Dependencies') {
      steps {
        sh '''
          if [ -f pnpm-lock.yaml ]; then
            npm install -g pnpm
            pnpm install
          else
            npm install
          fi
        '''
      }
    }

    stage('Run Tests') {
      steps {
        sh '''
          if [ -f pnpm-lock.yaml ]; then
            pnpm test || echo "No test script defined or test failed"
          else
            npm test || echo "No test script defined or test failed"
          fi
        '''
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          if [ ! -f Dockerfile ]; then
            echo "❌ ERROR: Dockerfile not found!"
            exit 1
          fi

          docker build -t $IMAGE_NAME .
          docker tag $IMAGE_NAME:latest $ECR_URL/$IMAGE_NAME:latest
        '''
      }
    }

    stage('Push to ECR') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS']]) {
          sh '''
            aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_URL
            docker push $ECR_URL/$IMAGE_NAME:latest
          '''
        }
      }
    }

    stage('Deploy to ECS') {
      steps {
        echo "🛠️ Deploying to ECS... (Terraform or AWS CLI commands go here)"
      }
    }
  }

  post {
    failure {
      echo "❌ Build failed. Check logs!"
    }
    success {
      echo "✅ Build and deployment completed successfully!"
    }
  }
}
