
---

# **Learner Report - MERN Application**

## **Project Overview**

The **Learner Report** is a MERN (MongoDB, Express.js, React.js, Node.js) application designed to manage learner reports. It consists of a frontend built with React.js, a backend API built using Node.js and Express.js, and a MongoDB database. The project is containerized using Docker and deployed on a Kubernetes cluster with Helm, which simplifies the deployment and management of the application components.

## **Table of Contents**

- [Project Overview](#project-overview)
- [Directory Structure](#directory-structure)
- [Technologies Used](#technologies-used)
- [Pre-requisites](#pre-requisites)
- [Local Development](#local-development)
- [Docker and Kubernetes Setup](#docker-and-kubernetes-setup)
- [Helm Setup](#helm-setup)
- [CI/CD Pipeline with Jenkins](#cicd-pipeline-with-jenkins)
- [Deployment Verification](#deployment-verification)

## **Directory Structure**

The repository has the following structure:

```
LEARNERREPORT/
│
├── backend/                    # Backend source code (Node.js + Express)
│   ├── Dockerfile              # Dockerfile for backend service
│   └── ...                     # Other backend files
│
├── frontend/                   # Frontend source code (React.js)
│   ├── Dockerfile              # Dockerfile for frontend service
│   └── ...                     # Other frontend files
│
├── k8s/                        # Kubernetes configurations
│   ├── backend/                # Backend Kubernetes manifests (deployment, service, secret, namespace)
│   ├── database/               # Database Kubernetes manifests (MongoDB deployment, service, PVC)
│   ├── frontend/               # Frontend Kubernetes manifests (deployment, service, namespace)
│   └── helm-package/           # Helm chart for deploying the whole application
│       ├── learnerreport/
│           ├── charts/         # Helm subcharts for backend, frontend, database
│           ├── templates/      # Templates for Kubernetes resources
│           ├── values.yaml     # Default values for Helm chart
│           └── Chart.yaml      # Helm chart metadata
│
├── docker-compose.yml          # Docker Compose file for building images locally
├── Jenkinsfile                 # Jenkins pipeline definition
└── README.md                   # This README file
```

## **Technologies Used**

- **Frontend**: React.js
- **Backend**: Node.js with Express.js
- **Database**: MongoDB
- **Containerization**: Docker
- **Orchestration**: Kubernetes (Minikube or a cloud Kubernetes cluster)
- **Package Management**: Helm
- **CI/CD**: Jenkins

## **Pre-requisites**

To run and deploy this application, you will need:

- Docker installed ([Install Docker](https://docs.docker.com/get-docker/))
- Kubernetes cluster (Minikube or cloud-based) ([Install Minikube](https://minikube.sigs.k8s.io/docs/start/))
- Helm installed ([Install Helm](https://helm.sh/docs/intro/install/))
- Jenkins installed and configured ([Install Jenkins](https://www.jenkins.io/doc/book/installing/))
- Access to Docker Hub or a private Docker registry for storing container images

## **Local Development**

### **1. Clone the repository**

```bash
git clone https://github.com/Ravikishans/learnerReport.git
cd learnerReport
```

### **2. Running the application locally**

You can run the application locally using Docker Compose.

#### **Step 1: Build the images**

```bash
docker-compose build
```

#### **Step 2: Run the containers**

```bash
docker-compose up
```

The application will be available at:

- Frontend: http://localhost:3000
- Backend API: http://localhost:3001
- MongoDB: Running on port 27017 inside the container

### **3. Stopping the containers**

To stop the running containers:

```bash
docker-compose down
```

## **Docker and Kubernetes Setup**

### **1. Build Docker Images**

You can build the Docker images for frontend and backend services using Docker Compose:

```bash
docker-compose build
```

### **2. Push Docker Images to a Registry**

You need to push the images to Docker Hub (or any other Docker registry):

```bash
docker login
docker-compose push
```

Make sure to configure the image repository names and tags in your `docker-compose.yml` file.

### **3. Deploy to Kubernetes**

#### **Step 1: Kubernetes Manifests**

Kubernetes manifest files for each component (backend, frontend, and database) are available in the `k8s/` directory. These YAML files define the deployment, service, and namespace configurations.

You can manually apply these manifests using `kubectl`:

```bash
kubectl apply -f k8s/backend/
kubectl apply -f k8s/frontend/
kubectl apply -f k8s/database/
```

#### **Step 2: Deploying with Helm**

Alternatively, you can deploy the entire application using Helm.

```bash
helm upgrade --install learnerreport ./k8s/helm-package/learnerreport --namespace default --create-namespace --debug
```

This will deploy all components (backend, frontend, and database) into your Kubernetes cluster.

## **Helm Setup**

The Helm chart is located in the `k8s/helm-package/learnerreport` directory. The chart includes subcharts for each component (backend, frontend, and database).

- `Chart.yaml`: Contains chart metadata.
- `values.yaml`: Contains configurable values for the Helm chart, including image repositories and tags.

You can customize the Helm chart values before deploying.

## **CI/CD Pipeline with Jenkins**

### **1. Jenkins Pipeline Overview**

The project uses a Jenkins pipeline defined in the `Jenkinsfile`. The pipeline performs the following steps:

- **Checkout code** from the GitHub repository.
- **Build Docker images** for the frontend and backend using Docker Compose.
- **Push Docker images** to the Docker registry.
- **Deploy the application** to a Kubernetes cluster using Helm.

### **2. Jenkinsfile Breakdown**

Here is a summary of the stages in the Jenkins pipeline:

1. **Checkout Code**: Clones the repository from GitHub.
2. **Build and Push Docker Images**: Builds the Docker images for the backend and frontend services and pushes them to the Docker registry.
3. **Deploy to Kubernetes**: Deploys the application using Helm to the Kubernetes cluster. Helm uses the Kubernetes credentials (kubeconfig) stored in Jenkins.
4. **Verify Deployment**: After deployment, it verifies the status of the application by listing the running pods.

### **3. Jenkinsfile**

```groovy
pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = "your-docker-registry"
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials')
        HELM_RELEASE_NAME = "learnerreport"
        HELM_CHART_PATH = "./k8s/helm-package/learnerreport"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Ravikishans/learnerReport'
            }
        }
        stage('Build and Push Docker Images') {
            steps {
                sh """
                    docker-compose build
                    docker login -u ${DOCKER_CREDENTIALS_USR} -p ${DOCKER_CREDENTIALS_PSW}
                    docker-compose push
                """
            }
        }
        stage('Deploy to Kubernetes using Helm') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                    sh """
                        helm upgrade --install ${HELM_RELEASE_NAME} ${HELM_CHART_PATH} \
                            --namespace default --create-namespace \
                            --kubeconfig=$KUBECONFIG --debug
                    """
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                sh "kubectl get pods -n ${BACKEND_NAMESPACE} --kubeconfig=$KUBECONFIG"
                sh "kubectl get pods -n ${FRONTEND_NAMESPACE} --kubeconfig=$KUBECONFIG"
                sh "kubectl get pods -n ${DATABASE_NAMESPACE} --kubeconfig=$KUBECONFIG"
            }
        }
    }
}
```

---

## **Jenkins Authentication Setup**

### **1. Setting Up Docker Credentials in Jenkins**

To push Docker images to your Docker Hub or private registry, you need to configure credentials in Jenkins.

#### **Step 1: Add Docker Credentials**

1. Go to Jenkins dashboard -> Manage Jenkins -> Manage Credentials.
2. Add new credentials:
   - **Username**: Your Docker Hub or registry username.
   - **Password**: Your Docker Hub or registry password.
   - **ID**: `dockerhub-credentials` (used in the Jenkinsfile).

#### **Step 2: Add Kubernetes Kubeconfig File as a Credential**

1. Save your Kubeconfig file (usually located at `~/.kube/config`) locally.

```bash
kubectl config view --raw
```
Copy and save it's output in Kubeconfig file.

2. Go to Jenkins dashboard -> Manage Jenkins -> Manage Credentials.
3. Add new credentials:
   - **Kind**: Secret file
   - **File**: Upload your Kubeconfig file.
   - **ID**: `kubeconfig-credentials`.

This Kubeconfig file will be used by Jenkins to access the Kubernetes cluster during the deployment process.

## **Deployment Verification**

Once the pipeline is executed, you can verify the deployment by running the following command:

```bash
kubectl get all --namespace=backendlr
kubectl get all --namespace=databaselr
kubectl get all --namespace=frontendlr
```

Ensure that the pods for the backend, frontend, and database are running successfully.

## **Troubleshooting**

### **Common Issues**

0. **Check for logs:**
    ```bash
    minikube logs
    ```

1. **Permission Denied for Minikube Cluster:**
   If you encounter permission errors like `unable to read client-key`, it could be related to file permissions. Ensure that the Helm process has access to the Kubeconfig file and all the necessary keys.
   
   **Fix:**
   ```
   sudo chmod 600 ~/.minikube/profiles/minikube/client.key
   sudo chmod 600 ~/.minikube/profiles/minikube/client.crt
   sudo chmod 600 ~/.minikube/ca.crt
   ```

2. **Kubernetes Cluster Unreachable:**
   If Helm or kubectl cannot reach the Kubernetes cluster, make sure your cluster is running

 and your kubeconfig is pointing to the correct cluster.

   **Fix:**
   ```
   kubectl config use-context <context-name>
   minikube start
   ```

3. **Docker Image Pull Failure:**
   If Kubernetes cannot pull Docker images, ensure the image is publicly available (if using Docker Hub) or that the Kubernetes cluster has access to your private Docker registry credentials.

   **Fix:**
   ```
   kubectl create secret docker-registry <secret-name> \
       --docker-server=<registry-server> \
       --docker-username=<username> \
       --docker-password=<password> \
       --docker-email=<email>
   ```

4. **Helm Installation Errors:**
   Helm installation might fail due to improper configuration or missing values.

   **Fix:**
   Double-check the `values.yaml` file in your Helm chart and ensure that all necessary values (image repository, tag, etc.) are properly defined.
---
