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

        stage('Deploy Backend to Kubernetes') {
            steps {
                script {
                    // Use kubeconfig from Jenkins credentials to authenticate
                    withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                        // Deploy backend using Helm
                        sh """
                            helm upgrade --install backend ${HELM_RELEASE_NAME}-backend ${HELM_CHART_PATH}/charts/backend \
                                --set image.repository=${DOCKER_REGISTRY}/learnerreport \
                                --set image.tag=backend \
                                --namespace ${BACKEND_NAMESPACE} \
                                --kubeconfig=$KUBECONFIG
                        """
                    }
                }
            }
        }

        stage('Deploy Frontend to Kubernetes') {
            steps {
                script {
                    // Deploy frontend using Helm
                    withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                        sh """
                            helm upgrade --install frontend ${HELM_RELEASE_NAME}-frontend ${HELM_CHART_PATH}/charts/frontend \
                                --set image.repository=${DOCKER_REGISTRY}/learnerreport \
                                --set image.tag=frontend \
                                --namespace ${FRONTEND_NAMESPACE} \
                                --kubeconfig=$KUBECONFIG
                        """
                    }
                }
            }
        }

        stage('Deploy Database to Kubernetes') {
            steps {
                script {
                    // Deploy database using Helm
                    withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                        sh """
                            helm upgrade --install database ${HELM_RELEASE_NAME}-database ${HELM_CHART_PATH}/charts/database \
                                --set image.repository=mongo \
                                --set image.tag=latest \
                                --namespace ${DATABASE_NAMESPACE} \
                                --kubeconfig=$KUBECONFIG
                        """
                    }
                }
            }
        }

        stage('Verify Deployments') {
            steps {
                script {
                    // Verify backend, frontend, and database deployments by listing pods in each namespace
                    sh "kubectl get pods -n ${BACKEND_NAMESPACE} --kubeconfig=$KUBECONFIG"
                    sh "kubectl get pods -n ${FRONTEND_NAMESPACE} --kubeconfig=$KUBECONFIG"
                    sh "kubectl get pods -n ${DATABASE_NAMESPACE} --kubeconfig=$KUBECONFIG"
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
