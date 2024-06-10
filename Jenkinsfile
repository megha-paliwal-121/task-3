pipeline {
    agent any
 
    environment {
        AWS_ACCOUNT_ID="979022608152"
        AWS_DEFAULT_REGION="us-west-2"
        BRANCH_NAME="main"
        IMAGE_REPO_NAME="megha_real-estate"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
 
    stages {
 
        stage('Logging into AWS ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                    sh "docker images -a -q | xargs docker rmi -f || true"
                }
 
            }
        }
 
 
// Building Docker images
        stage('Building image ') {
            steps{
                script {
                    dir('frontend'){
                    //env.git_commit_sha = sh(script: 'git rev-parse --short=6 HEAD', returnStdout: true).trim( )
                    sh "docker build -t frontend . "
                    }
                    dir('backend-fastify'){
                    env.git_commit_sha = sh(script: 'git rev-parse --short=6 HEAD', returnStdout: true).trim( )
                    sh "docker build -t backend . "
                    }
                }
            }
        }
 
// Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
            steps{
                script {
                    sh "docker push frontend"
                    sh "docker push backend"
                }
            }
        }
 
//Creating container
        stage('creating container for application') {
            steps{
                script {
                    
                    sh "ssh  ubuntu@54.203.188.224 /home/ubuntu/login-ecr.sh"
                    sh "ssh  ubuntu@54.203.188.224 sudo docker rm -f front || true"
                    sh "ssh  ubuntu@54.203.188.224 sudo docker rm -f backend || true"
                    sh "ssh  ubuntu@54.203.188.224 sudo docker images -a -q | xargs docker rmi -f || true"
                    sh "ssh  ubuntu@54.203.188.224 sudo docker run -itd --name front -p 4001:4200 --restart always frontend"
                    sh "ssh  ubuntu@54.203.188.224 sudo docker run -itd --name back -p 4002:8000 --restart always backend"

 
               }
            }
        }
    }
}
 
