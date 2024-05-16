pipeline{
    agent any
    environment{
        IMAGE_REPO_NAME="mani-devops"
        IMAGE_TAG="latest"
        REPOSITORY_URI = "969849892126.dkr.ecr.ap-south-1.amazonaws.com/${IMAGE_REPO_NAME}"
        SONARQUBE_SERVER = "http://98.70.58.127:9000"
        SONARQUBE_TOKEN = "8787534c7c916b53e53756f0f4de23e66b8badde"
        EC2_INSTANCE_IP="13.201.230.37"
        GITBRANCH_NAME="main"
        credentials_Id="jenkins-bitbucket"
        REPO_URL="https://github.com/Mani-tailwinds/mani-devops.git"
        ECR_URL="969849892126.dkr.ecr.ap-south-1.amazonaws.com"
        SSH_USERNAME="ubuntu"
        CONTAINER_NAME="condescending_gates"
        PORT_NUMBER="3003"
    }
    stages{
        stage('SCM checkout'){
            steps{
                checkout scmGit(branches: [[name: env.GITBRANCH_NAME]], extensions: [[$class: 'CloneOption', depth: 1]], userRemoteConfigs: [[credentialsId: env.credentials_Id, url: env.REPO_URL]])
            }
        }
    
        stage('Docker build'){
            steps{
                script{
                    sh 'docker build -t "${IMAGE_REPO_NAME}:${IMAGE_TAG}" .'
                    sh 'rm -f .env'
                }
            }
        }

 stage('ECR push'){
            steps{
                script{
                    sh"aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${ECR_URL}"
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:${IMAGE_TAG}"
                    sh "docker push ${ECR_URL}/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }
            }
        }
    }
}
