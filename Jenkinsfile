pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/shrustidodamani/Jenkinsandjava.git'
        AWS_REGION = 'ap-south-1'
        ECR_REPO_NAME = 'ecr-repo1'
        ECR_PUBLIC_REPO_URI = '092042970106.dkr.ecr.ap-south-1.amazonaws.com/ecr-repo1'
        IMAGE_TAG = 'latest'
        AWS_ACCOUNT_ID = '092042970106'
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
    }

    stages {
        stage('Install AWS CLI') {
            steps {
                script {
                    sh '''
                        set -e
                        echo "Installing AWS CLI..."
                        sudo apt update && sudo apt install unzip curl -y

                        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                        rm -rf aws
                        unzip -q awscliv2.zip
                        sudo ./aws/install --update
                        aws --version
                    '''
                }
            }
        }

        stage('Clone Repository') {
            steps {
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('Build') {
            steps {
                script {
                    sh '''
                        echo "Building Java application..."
                        mvn clean -B -Denforcer.skip=true package
                    '''
                }
            }
        }

         stage('Login to AWS ECR') {
            steps {
                script {
                sh '''
                    echo "Logging into ECR..."
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        echo "Building Docker image..."
                        docker build -t ${IMAGE_URI} .
                    '''
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    sh '''
                        echo "Pushing Docker image to ECR..."
                        docker push ${IMAGE_URI}
                    '''
                }
            }
        }

      
    }
    
    post {
        success {
            echo "Docker image pushed to ECR successfully and deployed."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
