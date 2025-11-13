pipeline {
    agent any
    environment{
        AWS_REGION = "eu-north-1"
        ECR_REPO = "160885294916.dkr.ecr.eu-north-1.amazonaws.com/nodejs-app"
        IMAGE_TAG ="latest"


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
                docker login --username AWS -password-stdin $ECR_REPO
                   '''
            }
        }
        stage('Tag & push Docker Image to ECR') {
            steps{
                sh '''
                docker tag nodejs-app:latest $ECR_REPO:$IMAGE_TAG
                docker push $ECR_REPO:$IMAGE_TAG        
                   '''
             }
        }
        stage('Deploy Container on EC2') {
            steps{
                sh '''
                docker stop nodejs-app || true
                docker rm nodejs-app || true
                docker pull $ECR_REPO:$IMAGE_TAG
                docker run -d -p 3000:3000 --name nodejs-app $ECR_REPO:$IMAGE_TAG
                   '''

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


