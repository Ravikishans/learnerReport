pipeline {
    agent any

    environment {
        // Docker Registry (adjust according to your Docker registry settings)
        DOCKER_REGISTRY = "ravikishans"
        DOCKER_CREDENTIALS = credentials('ravikishans')

        // Kubernetes namespaces
        BACKEND_NAMESPACE = "backendlr"
        FRONTEND_NAMESPACE = "frontendlr"
        DATABASE_NAMESPACE = "databaselr"

        // Helm release name
        HELM_RELEASE_NAME = "learnerreport"

        // Path to the Helm chart
        HELM_CHART_PATH = "./k8s/helm-package/learnerreport"
    }

    stages {

        stage('Checkout Code') {
            steps {
                script {
                    // Checkout code from GitHub repository
                    git branch: 'main', url: 'https://github.com/Ravikishans/learnerReport'
                }
            }
        }

        stage('Build and Push Docker Images') {
            steps {
                script {
                    // Build and push Docker images using docker-compose
                    sh """
                        docker-compose build
                        docker login -u ${DOCKER_CREDENTIALS_USR} -p ${DOCKER_CREDENTIALS_PSW}
                        docker-compose push
                    """
                }
            }
        }

        stage('Deploy to Kubernetes using Helm') {
            steps {
                script {
                    // Use kubeconfig from Jenkins credentials to authenticate with Kubernetes
                    withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                        // Helm install or upgrade command to deploy everything at once
                        sh """
                            helm upgrade --install ${HELM_RELEASE_NAME} ${HELM_CHART_PATH} \
                                --namespace default --create-namespace \
                                --kubeconfig=$KUBECONFIG --debug
                        """
                    }
                }
            }
        }

        stage('Verify Deployments') {
            steps {
                script {
                    // Verify backend, frontend, and database deployments by listing pods in each namespace
                    sh "kubectl get pods --namespace default --kubeconfig=$KUBECONFIG"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs for more details."
        }
    }
}
