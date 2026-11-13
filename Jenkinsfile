
pipeline {
    agent any

    environment {
        KUBECONFIG = '/var/jenkins_home/.kube/config'
        IMAGE_NAME = 'assignment7-app'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Source code fetched from GitHub'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Transfer Image to Minikube') {
            steps {
                sh '''
                    docker save -o assignment7-app.tar ${IMAGE_NAME}:${BUILD_NUMBER}
                    docker cp assignment7-app.tar minikube:/assignment7-app.tar
                    docker exec minikube ctr -n k8s.io images import /assignment7-app.tar
                    docker exec minikube rm -f /assignment7-app.tar
                    rm -f assignment7-app.tar
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    sed "s|assignment7-app:1|${IMAGE_NAME}:${BUILD_NUMBER}|g" k8s/deployment.yaml > k8s/deployment-ci.yaml
                    kubectl apply -f k8s/deployment-ci.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl rollout status deployment/assignment7-app --timeout=120s
                    kubectl get deployments
                    kubectl get pods
                    kubectl get services
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }
        failure {
            echo 'CI/CD Pipeline failed. Check the console output.'
        }
    }
}
