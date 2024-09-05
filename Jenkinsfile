pipeline {
    agent any

    environment {
        // Docker Registry
        DOCKER_REGISTRY = "ravikishans"
        DOCKER_CREDENTIALS = credentials('ravikishans')

        // Kubernetes and Helm Configuration
        BACKEND_NAMESPACE = "backendlr"
        FRONTEND_NAMESPACE = "frontendlr"
        DATABASE_NAMESPACE = "databaselr"
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

        stage('Build and Push Docker Images') {
            steps {
                script {
                    // Build and push images from docker-compose.yml
                    withCredentials([usernamePassword(credentialsId: 'ravikishans', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            docker-compose -f docker-compose.yml build
                            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                            docker-compose -f docker-compose.yml push
                        """
                    }
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                        // Deploy the application using Helm
                        sh """
                            helm upgrade --install ${HELM_RELEASE_NAME} ./k8s/helm-package/learnerreport \
                                --namespace default \
                                --set frontend.image.repository=${DOCKER_REGISTRY}/learnerreport \
                                --set frontend.image.tag=frontend \
                                --set backend.image.repository=${DOCKER_REGISTRY}/learnerreport \
                                --set backend.image.tag=backend \
                                --debug
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
