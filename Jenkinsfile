pipeline {
    agent any

    environment {
        DOCKER_IMAGE = '499756076901.dkr.ecr.us-east-1.amazonaws.com/demoapp:latest'
        DOCKER_CREDENTIALS_ID = 'your-dockerhub-credentials-id'
        KUBECONFIG_CREDENTIALS_ID = 'your-kubeconfig-credentials-id'
        EKS_CLUSTER_NAME = 'your-eks-cluster-name'
        region = 'us-east-1'
        accountID = '4997-5607-6901'
    }

    stages {
        stage('Build Docker image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${env.BUILD_ID}")
                }
            }
        }

        stage('Push Docker image') {
            steps {
                script {
                    // Authenticate to ECR
                    sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${accountID}.dkr.ecr.your-region.amazonaws.com"

                    // Tag the image with the ECR repository URL
                    // sh "docker tag ${DOCKER_IMAGE}:${env.BUILD_ID} your-account-id.dkr.ecr.your-region.amazonaws.com/your-repo-name:${env.BUILD_ID}"

                    // Push the image
                    sh "docker push ${DOCKER_IMAGE}:${env.BUILD_ID}"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
                    sh "kubectl config use-context ${EKS_CLUSTER_NAME}"
                    sh """
                        kubectl run your-deployment --image=${DOCKER_IMAGE}:${env.BUILD_ID} 
                        kubectl expose deployment your-deployment --type=LoadBalancer --port=80
                    """
                }
            }
        }
    }
}