# Deploy Django App to Kubernetes with GitHub Actions

This repository demonstrates how to automatically build and deploy a Django application to a Kubernetes cluster using GitHub Actions. The workflow is triggered by pushes to the `main` branch.

## Workflow Overview

The GitHub Actions workflow defined in this repository performs the following steps:

1. **Checkout the code**: Retrieves the code from the repository.
2. **Set up Docker Buildx**: Configures Docker Buildx for building multi-platform images.
3. **Get the Git commit SHA**: Obtains the latest commit SHA to tag the Docker image.
4. **Set environment variable**: Stores the Docker tag in an environment variable.
5. **Build Docker image**: Builds the Docker image for the Django application.
6. **Login to Docker Hub**: Authenticates to Docker Hub using stored credentials.
7. **Push Docker image**: Pushes the Docker image to Docker Hub.
8. **Update Kubernetes deployment**: Updates the Kubernetes deployment configuration with the new Docker image tag.
9. **Set up Kubectl**: Configures `kubectl` to interact with the Kubernetes cluster.
10. **Deploy to Kubernetes**: Applies the Kubernetes configurations to deploy the Django application and its dependencies.

## Workflow Configuration

Below is the GitHub Actions workflow configuration used in this repository:

```yaml
name: Deploy Django App to Kubernetes

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2
      with:
        ref: main

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Get Git Commit SHA
      id: get_version
      run: echo "::set-output name=sha::$(git rev-parse HEAD)"

    - name: Set environment variable
      run: echo "DOCK_TAG=${{ steps.get_version.outputs.sha }}" >> $GITHUB_ENV

    - name: Build Docker image
      run: docker build . -t 224574/django-todo:${{ env.DOCK_TAG }}

    - name: Login to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Push Docker image
      run: docker push 224574/django-todo:${{ env.DOCK_TAG }}

    - name: Update deployment with new image
      run: sed -i "s|tagname|${{ env.DOCK_TAG }}|" kubernetes/django-deployment.yaml

    - name: Set up Kubectl
      uses: azure/k8s-set-context@v1
      with:
          kubeconfig: ${{ secrets.KUBECONFIG }}

    - name: Deploy to Kubernetes
      run: |
        kubectl get pods
        kubectl apply -f kubernetes/postgress-pv.yaml
        kubectl apply -f kubernetes/postgres-deployment.yaml
        kubectl apply -f kubernetes/postgres-service.yaml
        kubectl apply -f kubernetes/django-deployment.yaml
        kubectl apply -f kubernetes/django-service.yaml
```

## Prerequisites

- **Docker Hub account**: Store your Docker images.
- **Kubernetes cluster**: Deploy your application.
- **GitHub Secrets**:
  - `DOCKER_USERNAME`: Your Docker Hub username.
  - `DOCKER_PASSWORD`: Your Docker Hub password.
  - `KUBECONFIG`: Base64 encoded Kubernetes configuration file.

## Setting Up GitHub Secrets


1. **Add secrets to your GitHub repository**:
   - Go to your repository on GitHub.
   - Click on `Settings`.
   - In the left sidebar, click on `Secrets and variables` and then `Actions`.
   - Add the following secrets:
     - `DOCKER_USERNAME`: Your Docker Hub username.
     - `DOCKER_PASSWORD`: Your Docker Hub password.
     - `KUBECONFIG`: The content of your `kubeconfig` file.

## How It Works

1. **Trigger**: The workflow is triggered by a push to the `main` branch.
2. **Code Checkout**: The repository code is checked out.
3. **Docker Setup**: Docker Buildx is set up for building the image.
4. **Git SHA**: The latest commit SHA is obtained and set as an environment variable.
5. **Docker Build and Push**: The Docker image is built and pushed to Docker Hub.
6. **Kubernetes Deployment**: The Kubernetes deployment is updated with the new Docker image and applied to the cluster.

By following these steps and using the provided workflow, you can automate the deployment of your Django application to a Kubernetes cluster using GitHub Actions.