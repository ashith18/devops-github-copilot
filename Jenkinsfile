pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'github-copilot-python'
        // KUBECONFIG_CREDENTIALS = credentials('KUBECONFIG_CREDENTIALS_ID')
        EKS_CLUSTER_NAME = 'my-cluster'
        region = 'us-east-1'
        accountID = '499756076901'
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
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_ID']]) {
                    script {
                        sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${accountID}.dkr.ecr.${region}.amazonaws.com"
                        sh "docker tag ${DOCKER_IMAGE}:${env.BUILD_ID} 499756076901.dkr.ecr.${region}.amazonaws.com/${DOCKER_IMAGE}:latest"
                        sh "docker push ${accountID}.dkr.ecr.${region}.amazonaws.com/${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }


        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_ID']]) {
                    withCredentials([file(credentialsId: 'KUBECONFIG', variable: 'KUBECONFIG')]) {
                        sh "kubectl apply -f k8s-manifest/namespace.yaml"
                        sh "kubectl apply -f k8s-manifest/deployment.yaml -n github-copilot "
                        sh "kubectl apply -f k8s-manifest/service.yaml -n github-copilot"
                        sh "kubectl apply -f k8s-manifest/hpa.yaml -n github-copilot"
                        sh "kubectl apply -f k8s-manifest/ingress.yaml -n github-copilot"
                    }
                }
                }
            }
        }
    }
