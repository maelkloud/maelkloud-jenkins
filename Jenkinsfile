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
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_credentials') {
                        def customImage = docker.build("${IMAGE_NAME}:${env.BUILD_NUMBER}", "--file Dockerfile .")
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_credentials') {
                        def customImage = docker.image("${IMAGE_NAME}:${env.BUILD_NUMBER}")
                        customImage.push()
                        customImage.push('latest')
                    }
                }
            }
        }

        stage('Terraform Init and Apply') {
            steps {
                script {
                    try {
                        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_CONTENT')]) {
                            writeFile file: 'kubeconfig.yaml', text: readFile(env.KUBECONFIG_CONTENT)
                            sh "terraform init"
                            echo 'Running Terraform init...'
                            sh 'terraform init'
                            echo 'Terraform init completed.'
    
                            echo "IMAGE_NAME: ${IMAGE_NAME}"
                            echo "BUILD_ID: ${env.BUILD_ID}"
                            echo "Docker image: ${IMAGE_NAME}:${env.BUILD_ID}"
    
                            echo 'Running Terraform apply...'
                            sh "terraform apply -auto-approve -var 'docker_image=${IMAGE_NAME}:${env.BUILD_ID}'"
                            echo 'Terraform apply completed.'
                        }
                    } catch (Exception err) {
                        echo "Error: ${err}"
                        currentBuild.result = 'FAILURE'
                        error('Terraform Init and Apply failed.')
                    }
                }
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
