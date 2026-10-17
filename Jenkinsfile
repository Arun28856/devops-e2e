pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub_creds')
        IMAGE_NAME = "lakshmanarun28/devops-e2e"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Arun28856/devops-e2e.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh "echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin"
                sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker push ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                        sed -i 's|<DOCKERHUB_USERNAME>/devops-app:latest|${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yaml
                        kubectl apply -f k8s/deployment.yaml
                        kubectl rollout status deployment/devops-e2e
                    """
                }
            }
        }
    }

    post {
        success { echo "Pipeline succeeded — image ${IMAGE_TAG} deployed." }
        failure { echo "Pipeline failed." }
    }
}