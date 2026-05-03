pipeline {
    agent any

    environment {
        DEPLOY_COLOR = "green"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Pulling code..."
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t myapp:${DEPLOY_COLOR} ./app/${DEPLOY_COLOR}"
                }
            }
        }

        stage('Load Image into Minikube') {
            steps {
                script {
                    sh "minikube image load myapp:${DEPLOY_COLOR}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "kubectl apply -f k8s/${DEPLOY_COLOR}-deployment.yaml"
                    sh "kubectl rollout status deployment/${DEPLOY_COLOR}-deployment"
                }
            }
        }

        stage('Switch Traffic') {
            steps {
                script {
                    sh """
                        kubectl patch service myapp-service \
                          -p '{"spec":{"selector":{"app":"myapp","version":"${DEPLOY_COLOR}"}}}'
                    """
                    echo "Traffic switched to: ${DEPLOY_COLOR}"
                }
            }
        }

        stage('Verify') {
            steps {
                sh "kubectl get pods -l version=${DEPLOY_COLOR}"
                sh "kubectl get service myapp-service"
            }
        }
    }

    post {
        success { echo "Deployment to ${DEPLOY_COLOR} SUCCESSFUL" }
        failure {
            echo "FAILED — rolling back to blue"
            sh """
                kubectl patch service myapp-service \
                  -p '{"spec":{"selector":{"app":"myapp","version":"blue"}}}'
            """
        }
    }
}
