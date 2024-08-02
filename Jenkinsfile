pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_FRONTEND_REPO = '058264319429.dkr.ecr.us-east-1.amazonaws.com/frontend'
        ECR_BACKEND_REPO = '058264319429.dkr.ecr.us-east-1.amazonaws.com/backend'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM',
                          userRemoteConfigs: [[url: 'https://github.com/Simulanis-Dev-Jagadeesha/my-new-project.git', credentialsId: 'gitcred']],
                          branches: [[name: '*/main']]])
            }
        }

        stage('Authenticate to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'awscred']]) {
                    script {
                        sh '''
                        echo "Configuring AWS CLI..."
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set region $AWS_REGION
                        
                        echo "Logging in to ECR..."
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_FRONTEND_REPO
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_BACKEND_REPO
                        '''
                    }
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Frontend Image') {
                    steps {
                        script {
                            echo "Building Frontend Docker image"
                            sh '''
                            cd frontend
                            docker build -t $ECR_FRONTEND_REPO:latest .
                            '''
                        }
                    }
                }

                stage('Build Backend Image') {
                    steps {
                        script {
                            echo "Building Backend Docker image"
                            sh '''
                            cd backend
                            docker build -t $ECR_BACKEND_REPO:latest .
                            '''
                        }
                    }
                }
            }
        }

        stage('Push Docker Images to ECR') {
            parallel {
                stage('Push Frontend Image') {
                    steps {
                        script {
                            echo "Pushing Frontend Docker image to ECR"
                            sh '''
                            docker tag $ECR_FRONTEND_REPO:latest $ECR_FRONTEND_REPO:latest
                            docker push $ECR_FRONTEND_REPO:latest
                            '''
                        }
                    }
                }

                stage('Push Backend Image') {
                    steps {
                        script {
                            echo "Pushing Backend Docker image to ECR"
                            sh '''
                            docker tag $ECR_BACKEND_REPO:latest $ECR_BACKEND_REPO:latest
                            docker push $ECR_BACKEND_REPO:latest
                            '''
                        }
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'awscred']]) {
                    script {
                        echo "Deploying to EKS"
                        sh '''
                        kubectl apply -f deployment.yml
                        kubectl apply -f service.yml
                        '''
                    }
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Please check the console output for more details.'
        }
    }
}
