# FastAPI Kubernetes Deployment with CI/CD

This guide provides a comprehensive walkthrough of deploying a simple FastAPI application to a Google Kubernetes Engine (GKE) cluster with a full CI/CD pipeline using GitHub Actions.

## 1. Prerequisites

Before you begin, ensure you have the following tools and accounts:

*   **Git:** To clone the repository and manage your code.
*   **Docker:** To build and run the Docker image locally.
*   **Google Cloud Platform (GCP) Account:** With billing enabled. You can create a free account [here](https://cloud.google.com/free).
*   **`gcloud` CLI:** The command-line tool for GCP. You can install it from [here](https://cloud.google.com/sdk/docs/install).
*   **`kubectl`:** The command-line tool for Kubernetes. You can install it from [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).
*   **Docker Hub Account:** To store the Docker image. You can create one [here](https://hub.docker.com/).
*   **GitHub Account:** To host the repository and use GitHub Actions.

## 2. Local Development

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/TestApp2619/CI-CD_fastapi_kubernetes.git
    cd CI-CD_fastapi_kubernetes
    ```

2.  **Create a virtual environment and install dependencies:**
    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    pip install -r requirements.txt
    ```

3.  **Run the application locally:**
    ```bash
    uvicorn main:app --reload
    ```
    The application will be available at `http://127.0.0.1:8000`.

## 3. Dockerization

1.  **Build the Docker image:**
    ```bash
    docker build -t fastapi-k8s:latest .
    ```

2.  **Run the Docker container:**
    ```bash
    docker run -p 8080:80 fastapi-k8s:latest
    ```
    The application will be available at `http://localhost:8080`.

## 4. Kubernetes Deployment (GKE)

### 4.1. Create a GKE Cluster

1.  **Create a new GCP project:**
    *   Go to the [GCP Console](https://console.cloud.google.com/) and create a new project.
    *   Set your project ID as an environment variable:
        ```bash
        export PROJECT_ID="your-project-id"
        gcloud config set project $PROJECT_ID
        ```

2.  **Enable the Kubernetes Engine API:**
    ```bash
    gcloud services enable container.googleapis.com
    ```

3.  **Create a new GKE cluster:**
    ```bash
    gcloud container clusters create-auto fastapi-cluster --region=us-central1
    ```
    This will create a new GKE cluster named `fastapi-cluster` in the `us-central1` region. This might take a few minutes.

### 4.2. Configure `kubectl`

1.  **Get the `kubeconfig` for the new cluster:**
    ```bash
    gcloud container clusters get-credentials fastapi-cluster --region=us-central1
    ```
    This command will automatically configure `kubectl` to connect to your new GKE cluster.

2.  **Verify the connection:**
    ```bash
    kubectl get nodes
    ```
    You should see a list of the nodes in your GKE cluster.

### 4.3. Deploy the Application to GKE

1.  **Apply the Kubernetes manifests:**
    ```bash
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    ```

2.  **Verify the deployment:**
    *   Check the pods:
        ```bash
        kubectl get pods
        ```
    *   Check the service:
        ```bash
        kubectl get service fastapi-service
        ```
        Wait for the `EXTERNAL-IP` of the service to be assigned. This might take a few minutes.

## 5. CI/CD with GitHub Actions

### 5.1. Set Up Secrets

1.  **Docker Hub:**
    *   Create a new access token on Docker Hub with "Read, Write, Delete" permissions.
    *   In your GitHub repository, go to "Settings" > "Secrets and variables" > "Actions" and create two new repository secrets:
        *   `DOCKERHUB_USERNAME`: Your Docker Hub username.
        *   `DOCKERHUB_TOKEN`: Your Docker Hub access token.

2.  **Kubernetes:**
    *   Generate a flattened `kubeconfig` with embedded credentials:
        ```bash
        kubectl config view --flatten
        ```
    *   Copy the entire output of the command.
    *   In your GitHub repository, create a new repository secret named `KUBECONFIG` and paste the content of the flattened `kubeconfig` as the value.

### 5.2. The Workflow

The `.github/workflows/main.yml` file defines the CI/CD pipeline. It has two jobs:

*   **`build-and-push`:** This job is triggered on every push to the `main` branch. It builds the Docker image and pushes it to your Docker Hub repository.
*   **`deploy`:** This job is triggered after the `build-and-push` job is successful. It connects to your GKE cluster using the `KUBECONFIG` secret and deploys the new image by applying the Kubernetes manifests.

## 6. Accessing the Application

Once the `deploy` job is successful, you can access your application using the external IP address of the `fastapi-service`.

1.  **Get the external IP address:**
    ```bash
    kubectl get service fastapi-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
    ```

2.  **Open the IP address in your browser.** You should see `{"Hello":"World"}`.

## 7. Clean Up

To avoid incurring charges on your GCP account, you should delete the resources you have created.

1.  **Delete the GKE cluster:**
    ```bash
    gcloud container clusters delete fastapi-cluster --region=us-central1
    ```

2.  **Delete the Docker image from Docker Hub.**
