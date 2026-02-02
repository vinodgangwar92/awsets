pipeline {
    agent any

    environment {
        // 🔐 AWS Credentials (Jenkins Credentials IDs)
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
        AWS_DEFAULT_REGION    = "ap-south-1"

        // EKS Cluster
        CLUSTER_NAME = "softapp-eks-cluster-2"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/vinodgangwar92/awsets.git'
            }
        }

        stage('Verify AWS Identity') {
            steps {
                bat 'aws sts get-caller-identity'
            }
        }

        stage('Terraform Init') {
            steps {
                dir('terraform') {
                    bat 'terraform init'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('terraform') {
                    bat 'terraform plan'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('terraform') {
                    bat 'terraform apply -auto-approve'
                }
            }
        }

        stage('Update kubeconfig') {
            steps {
                bat """
                aws eks update-kubeconfig ^
                --name %CLUSTER_NAME% ^
                --region %AWS_DEFAULT_REGION% ^
                --alias jenkins-eks
                """
            }
        }

        stage('Wait for Nodes (IMPORTANT)') {
            steps {
                retry(10) {
                    bat 'kubectl get nodes'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '''
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                '''
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD Pipeline completed successfully'
        }
        failure {
            echo '❌ Pipeline failed – check logs'
        }
    }
}
