pipeline {
    agent any
    environment{
        AWS_REGION = "eu-north-1"
        ECR_REGISTRY = "160885294916.dkr.ecr.eu-north-1.amazonaws.com"
        IMAGE_TAG    = "${GIT_COMMIT}"
        ECR_REPO = "nodejs-app"
        EC2_HOST = "ubuntu@13.60.213.97"
                 //<APP_EC2_PUBLIC_IP_OR_PRIVATE_IP>


    }
    stages{

        stage('Clean Workspace') {
           steps {
        cleanWs()
      }
        }
        stage('Checkout') {
            steps {
             git url: 'https://github.com/kushagra-g9/nodejs-sample-app.git' , branch: 'main'
            }
        }
       
        stage('Build Docker Image') {
            steps{
                sh 'docker build -t $ECR_REPO:latest .'

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
                docker tag $ECR_REPO:latest $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG
                docker push $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG        
                   '''
             }
        }
        stage('Deploy Container on EC2') {
            steps{
                sshagent(['app-ec2-ssh']) {
                sh '''
                ssh -o StrictHostKeyChecking=no $EC2_HOST "
                    sudo docker pull $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG &&
                    sudo docker stop nodejs-app || true &&
                    sudo docker rm nodejs-app || true &&
                    sudo docker run -d \
                        --restart always \
                        -p 3000:3000 \
                        --name nodejs-app \
                        $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG
                        "
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
    


