pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        IMAGE_NAME = "devops-demo"
        ECR_REPO   = "447407244516.dkr.ecr.ap-south-1.amazonaws.com/devops-demo"
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/Ezioraz/devops-cicd-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('AWS ECR Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set default.region ${AWS_REGION}

                    aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login --username AWS --password-stdin ${ECR_REPO}
                    '''
                }
            }
        }

        stage('Tag Image') {
            steps {
                sh 'docker tag ${IMAGE_NAME}:latest ${ECR_REPO}:latest'
            }
        }

        stage('Push to ECR') {
            steps {
                sh 'docker push ${ECR_REPO}:latest'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@<EC2_PUBLIC_IP> "
                        sudo docker pull ${ECR_REPO}:latest &&
                        sudo docker rm -f app || true &&
                        sudo docker run -d -p 5000:5000 --name app ${ECR_REPO}:latest
                    "
                    '''
                }
            }
        }
