pipeline {
    agent any

    environment {
        IMAGE = "devops-challenge-api:latest"
        NAMESPACE = "devops-challenge"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code is available in the workspace'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE} ./app
                '''
            }
        }

        stage('Load Image into Minikube') {
            steps {
                sh '''
                    docker save ${IMAGE} | docker exec -i minikube docker load
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/namespace.yaml
                    kubectl apply -f k8s/redis.yaml -n ${NAMESPACE}
                    kubectl apply -f k8s/api.yaml -n ${NAMESPACE}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl rollout status deployment/devops-api \
                        -n ${NAMESPACE} --timeout=120s

                    kubectl get pods -n ${NAMESPACE}
                    kubectl get services -n ${NAMESPACE}
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the stage logs for debugging.'
        }
    }
}
