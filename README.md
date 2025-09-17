# FastAPI Kubernetes Deployment Guide

This guide will walk you through deploying a simple FastAPI application to a local Kubernetes cluster using Minikube.

## Prerequisites

*   [Docker](https://docs.docker.com/get-docker/)
*   [Minikube](https://minikube.sigs.k8s.io/docs/start/)
*   [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
*   [uv](https://github.com/astral-sh/uv)

## 1. Set Up the Project

1.  **Clone the repository (or create the files manually):**

    ```bash
    git clone <repository-url>
    cd fastapi-k8s-deployment
    ```

2.  **Create a virtual environment and install dependencies:**

    ```bash
    uv venv
    source .venv/bin/activate
    uv pip install -r requirements.txt
    ```

## 2. Deploy to Minikube

1.  **Start Minikube:**

    ```bash
    minikube start
    ```

2.  **Build the Docker image:**

    ```bash
    docker build -t fastapi-k8s:latest .
    ```

3.  **Load the image into Minikube:**

    This makes the local image available to the Minikube cluster.

    ```bash
    minikube image load fastapi-k8s:latest
    ```

4.  **Apply the Kubernetes manifests:**

    This will create the Deployment and Service objects in Kubernetes.

    ```bash
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    ```

5.  **Verify the deployment:**

    Check that the pods are running:

    ```bash
    kubectl get pods
    ```

    You should see output similar to this:

    ```
    NAME                                  READY   STATUS    RESTARTS   AGE
    fastapi-deployment-8648cf9477-7g6bv   1/1     Running   0          3s
    fastapi-deployment-8648cf9477-j24xf   1/1     Running   0          3s
    fastapi-deployment-8648cf9477-vrqkg   1/1     Running   0          3s
    ```

6.  **Access the application:**

    Get the URL of the service:

    ```bash
    minikube service fastapi-service --url
    ```

    Open the returned URL in your browser. You should see `{"Hello":"World"}`.

## 3. Clean Up

To remove the resources created in this guide, run the following commands:

```bash
minikube service fastapi-service --url --disable
kubectl delete deployment fastapi-deployment
kubectl delete service fastapi-service
minikube stop
```
<!-- Trigger workflow -->
<!-- Trigger workflow again -->