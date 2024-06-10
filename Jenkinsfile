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
                    sh "echo y|docker system prune"
                    sh "docker images -a -q | xargs docker rmi -f || true"
                }
 
            }
        }
 
 
// Building Docker images
        stage('Building image ') {
            steps{
                script {
                    dir('frontend'){
                    env.git_commit_sha = sh(script: 'git rev-parse --short=6 HEAD', returnStdout: true).trim( )
                    sh "docker build -t ${REPOSITORY_URI}:${BRANCH_NAME}-${env.git_commit_sha}-frontend . "
                    }
                    dir('backend-fastify'){
                    env.git_commit_sha = sh(script: 'git rev-parse --short=6 HEAD', returnStdout: true).trim( )
                    sh "docker build -t ${REPOSITORY_URI}:${BRANCH_NAME}-${env.git_commit_sha}-backend . "
                    }
                }
            }
        }
 
// Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
            steps{
                script {
                    sh "docker push ${REPOSITORY_URI}:${BRANCH_NAME}-${env.git_commit_sha}-frontend"
                    sh "docker push ${REPOSITORY_URI}:${BRANCH_NAME}-${env.git_commit_sha}-backend"
                }
            }
        }
 
//Creating container
        stage('creating container for application') {
            steps{
                script {
                    
                    sh "ssh  ubuntu@54.203.188.224 /home/ubuntu/login-ecr.sh"
                    sh "ssh  ubuntu@54.203.188.224 sudo docker rm -f ${IMAGE_REPO_NAME}-${BRANCH_NAME} || true"
                    sh "ssh  ubuntu@54.203.188.224 sudo docker rm -f ${IMAGE_REPO_NAME}-${BRANCH_NAME}-B || true"
                    sh "ssh  ubuntu@54.203.188.224 sudo docker images -a -q | xargs docker rmi -f || true"
                    sh "ssh  ubuntu@54.203.188.224 sudo docker run -itd --name ${IMAGE_REPO_NAME}-${BRANCH_NAME} -p 4001:4200 --restart always ${REPOSITORY_URI}:${BRANCH_NAME}-${env.git_commit_sha}-frontend"
                    sh "ssh  ubuntu@54.203.188.224 sudo docker run -itd --name ${IMAGE_REPO_NAME}-${BRANCH_NAME}-B -p 4002:8000 --restart always ${REPOSITORY_URI}:${BRANCH_NAME}-${env.git_commit_sha}-backend"

 
               }
            }
        }
    }
}
 
