pipeline {
    agent any

    environment {
        // Docker Registry
        DOCKER_REGISTRY = "ravikishans"
        DOCKER_CREDENTIALS = credentials('ravikishans')
        
        // Kubernetes and Helm Configuration (if applicable)
        BACKEND_NAMESPACE = "backendlr"
        FRONTEND_NAMESPACE = "frontendlr"
        DATABASE_NAMESPACE = "databaselr"
        // KUBE_CONTEXT = "your-kube-context"  // if needed
        HELM_RELEASE_NAME = "learnerreport"
    }

    stages {

        stage('Checkout Code') {
            steps {
                script {
                    // Checkout the single repository where docker-compose.yml is located
                    git branch: 'main', url: 'https://github.com/Ravikishans/learnerReport'
                }
            }
        }

        stage('Build and Push Docker Images using Docker Compose') {
            steps {
                script {
                    // Build and push images from docker-compose.yml
                    sh """
                        docker-compose -f docker-compose.yml build
                        docker login -u ${DOCKER_CREDENTIALS_USR} -p ${DOCKER_CREDENTIALS_PSW}
                        docker-compose -f docker-compose.yml push
                    """
                }
            }
        }

        stage('Deploy with Helm (Optional)') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                        // Deploy the application using Helm (you can set specific image tags here)
                        sh """
                            // helm upgrade --install ${HELM_RELEASE_NAME} ./helm-chart \
                            //     --set frontend.image.repository=${DOCKER_REGISTRY}/frontend-image \
                            //     --set frontend.image.tag=latest \
                            //     --set backend.image.repository=${DOCKER_REGISTRY}/backend-image \
                            //     --set backend.image.tag=latest \
                            //     --namespace ${KUBE_NAMESPACE}
                            helm install ./k8s/helm-package/learnerreport --generate-name --namespace=default --debug
                        """
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Verify that the pods are running successfully
                    sh """
                        kubectl get pods -n ${BACKEND_NAMESPACE}
                        kubectl get pods -n ${FRONTEND_NAMESPACE}
                        kubectl get pods -n ${DATABASE_NAMESPACE}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for more information."
        }
    }
}
