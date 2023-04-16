// Jenkinsfile

pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub_credentials')
        KUBECONFIG_CREDS = credentials('minikube_cred')
        DOCKERHUB_USERNAME = '<DOCKER_HUB_USERNAME>'
        DOCKERHUB_PASSWORD = '<DOCKER_HUB_PASSWORD>'
        IMAGE_NAME = 'xmaeltht/python-webapp'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${env.BUILD_ID} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh 'docker login -u $DOCKERHUB_USER -p $DOCKERHUB_PASS'
                    sh 'docker push ${IMAGE_NAME}:${env.BUILD_ID}'
                }
            }
        }

        stage('Terraform Init and Apply') {
            steps {
                sh 'terraform init'
                sh 'terraform apply -auto-approve -var "docker_image=${IMAGE_NAME}:${env.BUILD_ID}"'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'minikube_cred', variable: 'KUBECONFIG_FILE')]) {
                    sh 'kubectl --kubeconfig=$KUBECONFIG_FILE apply -f k8s-deployment.yaml'
                }
            }
        }
    }
}
