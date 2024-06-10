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
        stage('Building image of backend') {
            steps{
                script {
                    env.git_commit_sha = sh(script: 'git rev-parse --short=6 HEAD', returnStdout: true).trim( )
                    sh "cd frontend/"
                    sh "docker build -t ${REPOSITORY_URI}:${BRANCH_NAME}-${env.git_commit_sha} . "
                    //sh "docker build -t ${REPOSITORY_URI}:${BRANCH_NAME}-${env.git_commit_sha} . -f backend-fastify/Dockerfile"
                }
            }
        }
 
// Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
            steps{
                script {
                    sh "docker push ${REPOSITORY_URI}:${BRANCH_NAME}-${env.git_commit_sha}"
                }
            }
        }
 
//Creating container
        stage('creating container for ladingpage') {
            steps{
                script {
                    
                    sh "ssh ec2-user@18.233.64.117 /home/ec2-user/login-ecr.sh"
                    sh "ssh  ec2-user@18.233.64.117 sudo docker rm -f ${IMAGE_REPO_NAME}-${BRANCH_NAME} || true"
                    sh "ssh  ec2-user@18.233.64.117 sudo docker images -a -q | xargs docker rmi -f || true"
                    sh "ssh  ec2-user@18.233.64.117 sudo docker run -itd --name ${IMAGE_REPO_NAME}-${BRANCH_NAME} -p 9090:80 --restart always ${REPOSITORY_URI}:${BRANCH_NAME}-${env.git_commit_sha}"


 
               }
            }
        }
    }
}
 
