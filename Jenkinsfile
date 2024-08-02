pipeline {
    agent any
    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO_FRONTEND = '058264319429.dkr.ecr.us-east-1.amazonaws.com/node-todo-app/frontend'
        ECR_REPO_BACKEND = '058264319429.dkr.ecr.us-east-1.amazonaws.com/node-todo-app/backend'
    }
    stages {
        stage('Authenticate to ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'awscred', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    script {
                        sh '''
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO_FRONTEND
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO_BACKEND
                        '''
                    }
                }
            }
        }
        stage('Build Docker Images') {
            parallel {
                stage('Build Frontend Image') {
                    steps {
                        dir('frontend') {
                            script {
                                sh '''
                                docker build -t $ECR_REPO_FRONTEND:latest -f Dockerfile .
                                '''
                            }
                        }
                    }
                }
                stage('Build Backend Image') {
                    steps {
                        dir('backend') {
                            script {
                                sh '''
                                docker build -t $ECR_REPO_BACKEND:latest -f Dockerfile .
                                '''
                            }
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
                            sh '''
                            docker push $ECR_REPO_FRONTEND:latest
                            '''
                        }
                    }
                }
                stage('Push Backend Image') {
                    steps {
                        script {
                            sh '''
                            docker push $ECR_REPO_BACKEND:latest
                            '''
                        }
                    }
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                script {
                    // Assuming you have deployment configurations in YAML files
                    sh '''
                    kubectl apply -f deployment.yml
                    kubectl apply -f service.yml
                    '''
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
