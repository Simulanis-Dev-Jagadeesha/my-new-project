pipeline {
    agent any

    environment {
        AWS_CREDENTIALS = credentials('awscred')
        FRONTEND_REPO_URL = "https://github.com/Simulanis-Dev-Jagadeesha/my-new-project.git"
        BACKEND_REPO_URL = "https://github.com/Simulanis-Dev-Jagadeesha/my-new-project.git"
        ECR_REPO_URI_FRONTEND = "058264319429.dkr.ecr.us-east-1.amazonaws.com/frontend"
        ECR_REPO_URI_BACKEND = "058264319429.dkr.ecr.us-east-1.amazonaws.com/backend"
    }

    stages {
        stage('Clone Frontend Repository') {
            steps {
                dir('frontend') {
                    git credentialsId: 'gitcred', url: "${env.FRONTEND_REPO_URL}", branch: 'main'
                }
            }
        }
        stage('Clone Backend Repository') {
            steps {
                dir('backend') {
                    git credentialsId: 'gitcred', url: "${env.BACKEND_REPO_URL}", branch: 'main'
                }
            }
        }
        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    script {
                        dockerImageFrontend = docker.build("${env.ECR_REPO_URI_FRONTEND}:latest")
                    }
                }
            }
        }
        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    script {
                        dockerImageBackend = docker.build("${env.ECR_REPO_URI_BACKEND}:latest")
                    }
                }
            }
        }
        stage('Authenticate to ECR') {
            steps {
                script {
                    sh '''
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 058264319429.dkr.ecr.us-east-1.amazonaws.com
                    '''
                }
            }
        }
        stage('Push Frontend to ECR') {
            steps {
                script {
                    docker.withRegistry("https://${env.ECR_REPO_URI_FRONTEND}", "${env.AWS_CREDENTIALS}") {
                        dockerImageFrontend.push('latest')
                    }
                }
            }
        }
        stage('Push Backend to ECR') {
            steps {
                script {
                    docker.withRegistry("https://${env.ECR_REPO_URI_BACKEND}", "${env.AWS_CREDENTIALS}") {
                        dockerImageBackend.push('latest')
                    }
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                script {
                    sh """
                    aws eks update-kubeconfig --name my-cluster1 --region us-east-1
                    kubectl apply -f deployment.yml
                    kubectl apply -f service.yml
                    """
                }
            }
        }
    }
}
