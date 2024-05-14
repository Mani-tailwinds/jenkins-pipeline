pipeline{
    agent any
    environment{
        IMAGE_REPO_NAME="kishore-devops"
        IMAGE_TAG="latest"
        REPOSITORY_URI = "969849892126.dkr.ecr.eu-north-1.amazonaws.com/${IMAGE_REPO_NAME}"
        SONARQUBE_SERVER = "http://98.70.58.127:9000"
        SONARQUBE_TOKEN = "8787534c7c916b53e53756f0f4de23e66b8badde"
        EC2_INSTANCE_IP="13.51.106.20"
        GITBRANCH_NAME="main"
        credentials_Id="jenkins-bitbucket"
        REPO_URL="https://github.com/Mani-tailwinds/jenkins-pipeline.git"
        ECR_URL="969849892126.dkr.ecr.eu-north-1.amazonaws.com"
        SSH_USERNAME="ubuntu"
        CONTAINER_NAME="condescending_gates"
        PORT_NUMBER="3003"
        AWS_ACCESS_KEY_ID ="AKIA6DT4O5UPNBURBYFR"
        AWS_SECRET_ACCESS_KEY ="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"
        AWS_DEFAULT_REGION = "eu-north-1"
    }
    stages{
        stage('SCM checkout'){
            steps{
                checkout scmGit(branches: [[name: env.GITBRANCH_NAME]], extensions: [[$class: 'CloneOption', depth: 1]], userRemoteConfigs: [[credentialsId: env.credentials_Id, url: env.REPO_URL]])
            }
        }
      stage('SCM scan') {
            steps {
                echo "------------------ Sonarqube analysis ----------------------"
                 script {
                     def scannerHome = tool name: 'jenkins-sonarqube-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                      withSonarQubeEnv(credentialsId: 'jenkins-sonarqube') {
                          sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=${IMAGE_REPO_NAME} \
                            -Dsonar.host.url=${env.SONARQUBE_SERVER} \
                            -Dsonar.login=${env.SONARQUBE_TOKEN}
                            """
                        }
                    }
            }
        }
        // stage('Secrets scan'){
        //     steps{
        //         echo "------------------ Gitleaks analysis ----------------------"
        //         script{
        //             try{
        //                 sh '/gitleaks/gitleaks detect --source=. --report-path=./.gitleaks-report.json'
        //             } catch (Exception e){
        //                 echo "Gitleaks scan completed with errors, but the pipeline will continue"
        //             }
        //         }
        //     }
        // }
        //stage('Dependency scan'){
        //    steps{
        //        echo "------------------ OWASP analysis ----------------------"
        //        script{
        //            sh 'npm install'
        //        }
        //        dependencyCheck additionalArguments: '--format HTML --out .dependency-check-report.html', odcInstallation: 'Dependent-CVE'
        //    }
        //}
        stage('env update'){
            steps{
                sshagent(['ssh-agent-cred']){
                    // sh "rsync -avz ${SSH_USERNAME}@${EC2_INSTANCE_IP}:/home/ubuntu/prodConfig/ippopay-lending-service/.env  ./.env"
                    sh "ssh -o StrictHostKeyChecking=no ${env.SSH_USERNAME}@${env.EC2_INSTANCE_IP} sudo rm -rf /opt/test "
                    sh "ssh -o StrictHostKeyChecking=no ${env.SSH_USERNAME}@${env.EC2_INSTANCE_IP} sudo mkdir /opt/test "
                }
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
        // stage('Docker scan'){
        //     steps{
        //         echo "------------------ Trivy analysis ----------------------"
        //         script{
        //             sh 'trivy "${IMAGE_REPO_NAME}:${IMAGE_TAG}"'
        //         }
        //     }
        // }
        stage('ECR push'){
            steps{
                script{
                    sh"aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin ${ECR_URL}"
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:${IMAGE_TAG}"
                    sh "docker push ${ECR_URL}/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }
            }
        }
       stage('SSH deploy') {
           steps {
               sshagent(['ssh-agent-cred']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${SSH_USERNAME}@${EC2_INSTANCE_IP}
                          # Docker login to ECR
                          $(aws ecr get-login --no-include-email --region ${AWS_DEFAULT_REGION})
                          # Remove old container if exists
                          sudo docker rm -f ${CONTAINER_NAME} || true
                          # Remove old image if exists
                          sudo docker rmi ${REPOSITORY_URI}:${IMAGE_TAG} || true
                       '''
               }
           }
         }
        //    steps{
        //        script{
        //            hangoutsNotify message: "Job ${IMAGE_REPO_NAME} is completed.",tokencredentialsId: 'chat-notification',threadByJob: false
        //        }
        //    }
        //}
        // stage('Post build'){
        //     steps{
        //         script{
        //             cleanWs deleteDirs: true, notFailBuild: true, patterns: [[pattern: '.dependency-check-report.html', type: 'EXCLUDE'], [pattern: '.gitleaks-report.json', type: 'EXCLUDE']]
        //         }
        //     }
        // }
    }
}
