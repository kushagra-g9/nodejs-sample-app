pipeline {
    agent any
    environment{
        AWS_REGION = "eu-north-1"
        ECR_REGISTRY = "160885294916.dkr.ecr.eu-north-1.amazonaws.com"
        IMAGE_TAG = "latest"
        ECR_REPO = "nodejs-app"


    }
    stages{
        stage('Checkout') {
            steps {
             git url: 'https://github.com/kushagra-g9/nodejs-sample-app.git' , branch: 'main'
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'

            }
        }
        stage('Build Docker Image') {
            steps{
                sh 'docker build -t nodejs-app .'

            }
        }
        stage('Login to ECR (Passwordless)') {
            steps{
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $ECR_REGISTRY
                   '''
            }
        }
        stage('Tag & push Docker Image to ECR') {
            steps{
                sh '''
                docker tag nodejs-app:latest $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG
                docker push $ECR_REGISTRTY/$ECR_REPO:$IMAGE_TAG        
                   '''
             }
        }
        stage('Deploy Container on EC2') {
            steps{
                sh '''
                docker stop nodejs-app || true
                docker rm nodejs-app || true
                docker pull $ECR_REGISTRTY/$ECR_REPO:$IMAGE_TAG
                docker run -d -p 3000:3000 --name nodejs-app $ECR_REGISTRTY/$ECR_REPO:$IMAGE_TAG
                   '''

            }
        }
    }
        post {
            success {
                echo "Deployment completed successfully!"
            }
            failure {
                echo "Pipeline failed."
            }
        }

     }
    


