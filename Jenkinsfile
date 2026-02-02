pipeline {
    agent any

    environment {
        IMAGE_NAME = "vinodgangwar92/etslab"
        IMAGE_TAG  = "123"
    }

    stages {

        stage('Checkout Code from GitLab') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/vinodgangwar92/ets.git'
            }
        }

        stage('Docker Build') {
            steps {
                bat '''
                docker build -t %IMAGE_NAME%:%IMAGE_TAG% .
                '''
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat '''
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    docker push %IMAGE_NAME%:%IMAGE_TAG%
                    '''
                }
            }
        }

        stage('Update Kubernetes Image in YAML') {
            steps {
                powershell '''
                $image = "$env:IMAGE_NAME`:$env:IMAGE_TAG"
                (Get-Content deployment.yaml) `
                  -replace "IMAGE_NAME", $image |
                Set-Content deployment.yaml
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    bat '''
                    set KUBECONFIG=%KUBECONFIG%

                    kubectl get nodes || exit /b 1

                    kubectl apply -f deployment.yaml || exit /b 1
                    kubectl apply -f service.yaml || exit /b 1
                    '''
                }
            }
        }

    }
}
