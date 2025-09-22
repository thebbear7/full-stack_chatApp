pipeline {
    agent any

    environment {
        NAMESPACE = "chat-app"
        BACKEND_IMAGE = "mahbeer/chatapp-backend:latest"
        FRONTEND_IMAGE = "mahbeer/chatapp-frontend:latest"
        AWS_REGION = "ap-south-1"
        EKS_CLUSTER = "chat-cluster"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-creds', url: 'https://github.com/thebbear7/full-stack_chatApp.git'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo "$DOCKER_PASS" | docker login --username "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Build & Push Backend Image') {
            steps {
                sh """
                    docker buildx build --platform linux/amd64 -t mahbeer/chatapp-backend:latest ./backend --push

                """
            }
        }

        stage('Build & Push Frontend Image') {
            steps {
                sh """
                    docker buildx build --platform linux/amd64 -t $FRONTEND_IMAGE ./frontend --push
                """
            }
        }

        stage('Configure kubectl for EKS') {
            steps {
                withAWS(credentials: 'aws-creds-id', region: 'us-east-1') {
                    sh """
                aws eks update-kubeconfig --name your-eks-cluster-name
            """
        }
    }
}


        stage('Deploy Backend') {
            steps {
                sh """
                    kubectl apply -n $NAMESPACE -f k8s/backend-deployment.yaml
                    kubectl rollout status deployment/backend-deployment -n $NAMESPACE
                """
            }
        }

        stage('Deploy Frontend') {
            steps {
                sh """
                    kubectl apply -n $NAMESPACE -f k8s/frontend-deployment.yaml
                    kubectl rollout status deployment/frontend-deployment -n $NAMESPACE
                """
            }
        }
    }

}
