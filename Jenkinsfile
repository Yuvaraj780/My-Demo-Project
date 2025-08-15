pipeline {
    agent {label 'master'}
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
                git 'https://github.com/Yuvaraj780/My-Demo-Project.git'
            }
        }
        stage('CODE-BUILD') {
            steps {
                //sh 'whoami'
                sh 'mvn clean install'
            }
        }
        stage('Docker Build & Push to ECR') {
            steps {
                script {
                    echo "Building Docker image and pushing to ECR..."
                    withAWS(region: "${env.AWS_REGION}", credentials: 'aws') {
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
                    echo "Building Docker image and pushing to ECR..."
                    withAWS(region: "${env.AWS_REGION}", credentials: 'aws') {
                      sh 'whoami'
                       sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin ${ECR_REPO.split('/')[0]}"
                       sh 'docker pull $ECR_REPO:latest'
                       sh 'docker stop con-1'
                       sh 'docker rm con-1'
                       sh 'docker run -itd --name con-1 -p "81:8080" $ECR_REPO:latest '
                    }
                }
            }
        }
        
    }
}

