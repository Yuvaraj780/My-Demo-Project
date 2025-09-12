pipeline {
    agent { label 'master' }
    tools {
        maven 'mvn'
    }
    triggers {
        githubPush()
    }
    
    environment {
        SONARQUBE_ENV = 'sonar'
        ECR_REPO = '578294758767.dkr.ecr.us-east-1.amazonaws.com/my-img'
        AWS_REGION = 'us-east-1'
        IMAGE_NAME = 'my-img'
    }
    
    stages {
        stage('CODE-PULL') {
            steps {
                git 'https://github.com/iam-yuvi/My-Demo-Project.git'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo "Starting SonarQube static code analysis..."
                withSonarQubeEnv("${env.SONARQUBE_ENV}") {
                    withCredentials([string(credentialsId: 'sonar-id', variable: 'SONAR_TOKEN')]) {
                        sh """
                          mvn clean verify sonar:sonar \
                              -Dsonar.projectKey=demo-prj \
                              -Dsonar.projectName='demo-prj' \
                              -Dsonar.host.url=http://18.212.234.111:9000 \
                              -Dsonar.token=$SONAR_TOKEN
                        """
                    }
                }
            }
        }
        
        stage('CODE-BUILD') {
            steps {
                sh 'whoami'
                sh 'mvn clean install'
            }
        }
        
        stage('Docker Build & Push to ECR') {
            steps {
                script {
                    echo "Building Docker image and pushing to ECR..."
                    withAWS(region: "${env.AWS_REGION}", credentials: 'aws-creds') {
                        sh 'whoami'
                        sh 'docker build -t $IMAGE_NAME .'
                        sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin ${ECR_REPO.split('/')[0]}"
                        sh 'docker tag $IMAGE_NAME:latest $ECR_REPO:latest'
                        sh 'docker push $ECR_REPO:latest'
                    }
                }
            }
        }
        
        stage('Deploy') {
            agent { label 'ECR-Slave'}
            steps {
                script {
                    echo "Deploying container from ECR..."
                    withAWS(region: "${env.AWS_REGION}", credentials: 'aws-creds') {
                        sh 'whoami'
                        sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin ${ECR_REPO.split('/')[0]}"
                        sh 'docker pull $ECR_REPO:latest'
                        sh 'docker stop con-1 || true'
                        sh 'docker rm con-1 || true'
                        sh 'docker run -itd --name con-1 -p "81:8080" $ECR_REPO:latest'
                    }
                }
            }
        }

        // FOR DEPLOYING INTO EKS CLUSTER!

             // stage('DEPLOY-TO-EKS') {
                    //     steps {
                    //         script {
                    //             withAWS(region: env.AWS_REGION, credentials: 'aws-creds') {
                    //                 echo "Updating kubeconfig for EKS cluster..."
                    //                 sh """
                    //                   aws eks update-kubeconfig --name ${env.CLUSTER_NAME} --region ${env.AWS_REGION}
                    //                 """
                                    
                    //                 echo "Deploying new image to EKS..."
                    //                 sh """
                    //                   kubectl set image deployment/${env.DEPLOYMENT_NAME} \
                    //                   my-container=${env.ECR_REPO}:latest \
                    //                   -n ${env.K8S_NAMESPACE}
                    //                 """
            
                    //             }
                    //         }
                    //     }
                    // }
                    
    }  
    
    post {
        success {
            emailext (
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Good news!</p>
                         <p>Job <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> finished successfully.</p>
                         <p>Check console: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>""",
                to: 'yuviyuvaraj7639@gmail.com'
            )
        }
        failure {
            emailext (
                subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Job <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> has failed.</p>
                         <p>Check console: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>""",
                to: 'yuviyuvaraj7639@gmail.com'
            )
        }
    }
}
