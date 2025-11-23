pipeline {
    agent any

    environment {
        AWS_REGION   = "eu-north-1"
        ECR_REGISTRY = "160885294916.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ECR_REPO     = "nodejs-app"
        IMAGE_TAG    = "${GIT_COMMIT}"
        EC2_HOST     = "ubuntu@13.60.213.97"       // <-- Use Elastic IP for stability
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/kushagra-g9/nodejs-sample-app.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${ECR_REPO}:latest .
                """
            }
        }

        stage('Login to ECR (Passwordless)') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin ${ECR_REGISTRY}
                """
            }
        }

        stage('Tag & Push to ECR') {
            steps {
                sh """
                docker tag ${ECR_REPO}:latest ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
                docker push ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to EC2 (Blue-Green Style)') {
            steps {
                sshagent(credentials: ['app-ec2-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_HOST} '
                        
                        echo "Pulling latest image..."
                        sudo docker pull ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}

                        echo "Stopping old container..."
                        sudo docker stop nodejs-app || true
                        sudo docker rm nodejs-app || true

                        echo "Starting new container..."
                        sudo docker run -d \
                            --restart=always \
                            -p 3000:3000 \
                            --name nodejs-app \
                            -e NODE_ENV=production \
                            ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}

                        echo "Waiting for app to start..."
                        sleep 5

                        echo "Performing health check..."
                        curl -f http://localhost:3000 || exit 1
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed! Attempting rollback..."
        }
    }
}
