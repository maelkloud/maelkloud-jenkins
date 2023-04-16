// Jenkinsfile

pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub-credentials')
        KUBECONFIG_CREDS = credentials('kubeconfig-credentials')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t your-dockerhub-username/python-webapp:${env.BUILD_ID} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh 'docker login -u $DOCKERHUB_USER -p $DOCKERHUB_PASS'
                    sh 'docker push your-dockerhub-username/python-webapp:${env.BUILD_ID}'
                }
            }
        }

        stage('Terraform Init and Apply') {
            steps {
                sh 'terraform init'
                sh 'terraform apply -auto-approve -var "docker_image=your-dockerhub-username/python-webapp:${env.BUILD_ID}"'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG_FILE')]) {
                    sh 'kubectl --kubeconfig=$KUBECONFIG_FILE apply -f k8s-deployment.yaml'
                }
            }
        }
    }
}
