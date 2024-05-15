pipeline{
    agent any
    environment{
        IMAGE_REPO_NAME="stage-ippopay-lending-service"
        IMAGE_TAG="latest"
        REPOSITORY_URI = "475157955544.dkr.ecr.ap-south-1.amazonaws.com/${IMAGE_REPO_NAME}"
        SONARQUBE_SERVER = "https://sam-cicd.centralindia.cloudapp.azure.com/sonar"
        SONARQUBE_TOKEN = "sqa_d035564f755a57a2f237ef46d1b73fa16aa2451f"
        EC2_INSTANCE_IP="ec2-3-10-238-48.ap-south-1.compute.amazonaws.com"
        GITBRANCH_NAME=“*/dev"
        credentials_Id="jenkins-bitbucket"
        REPO_URL="https://github.com/Mani-tailwinds/jenkins-pipeline.git"
        ECR_URL="475157955544.dkr.ecr.ap-south-1.amazonaws.com"
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
        stage('SCM scan'){
             steps{
                echo "------------------ Sonarqube analysis ----------------------"
                script{
                    def scannerHome = tool name: 'jenkins-sonarqube-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube'){
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=${IMAGE_REPO_NAME} \
                            -Dsonar.host.url="${env.SONARQUBE_SERVER}" \
                            -Dsonar.login="${env.SONARQUBE_TOKEN}"
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
                sshagent(['ippopay_staging']){
                    sh "rsync -avz ${SSH_USERNAME}@${EC2_INSTANCE_IP}:/home/ubuntu/prodConfig/ippopay-lending-service/.env  ./.env"
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
                    sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${ECR_URL}'
                    sh 'docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:${IMAGE_TAG}'
                    sh 'docker push ${ECR_URL}/${IMAGE_REPO_NAME}:${IMAGE_TAG}'
                }
            }
        }
        stage('SSH deploy'){
            steps{
                sshagent(['ippopay_staging']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${SSH_USERNAME}@${EC2_INSTANCE_IP}
                            #remove old container
                            sudo docker stop ${CONTAINER_NAME}
                            sudo docker rm -f ${CONTAINER_NAME}
                            sudo docker rmi ${REPOSITORY_URI}:${IMAGE_TAG}
                            # Within the EC2 instance, execute the AWS ECR login
                            sudo aws ecr get-login-password --region ap-south-1 | sudo docker login --username AWS --password-stdin ${ECR_URL}
                            # Run the Docker image
                            sudo docker run --log-driver json-file --log-opt tag="{{.ImageName}}|{{.Name}}|{{.ImageFullID}}|{{.FullID}}"  --log-opt max-size=20m --log-opt max-file=5 -e APP_ENV=staging -d --name '${CONTAINER_NAME}' --restart "on-failure" -p ${PORT_NUMBER}:${PORT_NUMBER} ${ECR_URL}/${IMAGE_REPO_NAME}:${IMAGE_TAG}'
                    """
                }
            }
        }
        //stage('Build notification'){
        //    steps{
        //        script{
        //            hangoutsNotify message: "Job ${IMAGE_REPO_NAME} is completed.",tokencredentialsId: 'chat-notification',threadByJob: false
        //        }
        //    }
        //}
        stage('Post build'){
            steps{
                script{
                    cleanWs deleteDirs: true, notFailBuild: true, patterns: [[pattern: '.dependency-check-report.html', type: 'EXCLUDE'], [pattern: '.gitleaks-report.json', type: 'EXCLUDE']]
                }
            }
        }
    }
}






















