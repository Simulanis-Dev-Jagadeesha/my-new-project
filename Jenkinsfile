pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        CLUSTER_NAME = 'my-cluster1'
        ECR_FRONTEND_REPO = '058264319429.dkr.ecr.us-east-1.amazonaws.com/frontend'
        ECR_BACKEND_REPO = '058264319429.dkr.ecr.us-east-1.amazonaws.com/backend'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                script {
                    echo "Checking out code from Git repository"
                    checkout([$class: 'GitSCM',
                              userRemoteConfigs: [[url: 'https://github.com/Simulanis-Dev-Jagadeesha/my-new-project.git', credentialsId: 'gitcred']],
                              branches: [[name: '*/main']]])
                }
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
                        dir('frontend') {
                            script {
                                sh 'docker build -t $ECR_FRONTEND_REPO:latest -f Dockerfile .'
                            }
                        }
                    }
                }

                stage('Build Backend Image') {
                    steps {
                        dir('backend') {
                            script {
                                sh 'docker build -t $ECR_BACKEND_REPO:latest -f Dockerfile .'
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
                            sh 'docker push $ECR_FRONTEND_REPO:latest'
                        }
                    }
                }

                stage('Push Backend Image') {
                    steps {
                        script {
                            sh 'docker push $ECR_BACKEND_REPO:latest'
                        }
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    // Ensure the cluster is reachable
                    sh '''
                    echo "Updating kubeconfig for cluster..."
                    aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
                    
                    echo "Checking cluster connectivity..."
                    kubectl cluster-info
                    if [ $? -ne 0 ]; then
                        echo "Failed to connect to the cluster. Exiting."
                        exit 1
                    fi
                    '''

                    // Apply Kubernetes configurations
                    sh 'kubectl apply -f deployment.yml'
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
