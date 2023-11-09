# Step-by-Step Guide for Deploying a Container on a VM Server with GitHub Actions

## 1. Prepare Your Project

Ensure your project has a Dockerfile that defines how to build your container. Make sure your containerized application exposes the necessary ports.

## 2. Set Up Your VM Server

Make sure your VM server is running and accessible over the network. Install Docker on your VM server so that it can run containers. Ensure the necessary ports are open on your VM server.

## 3. Generate SSH Key Pair

On your local machine, generate an SSH key pair using the `ssh-keygen` command. Add the public key (`id_rsa.pub`) to the authorized keys on your VM server.

## 4. Create GitHub Actions Workflow

In your GitHub repository, create a new directory named `.github/workflows`. Inside this directory, create a YAML file (e.g., `deploy.yml`) for your workflow.

## 5. Define Workflow Steps

Define the workflow, starting with the trigger event (e.g., push to the main branch). Add a step to check out your repository. Configure the environment variables for your VM server's SSH connection.

```yaml
name: Deploy to VM

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      - name: Set up SSH
        uses: webfactory/ssh-agent@v0.5.3
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
```

## 6. Configure Secrets
In your GitHub repository, go to Settings > Secrets. Add a secret named SSH_PRIVATE_KEY and paste the private key generated in step 3.

## 7. Add Deployment Steps
Add steps to build your Docker image and push it to a container registry (e.g., Docker Hub).

```
      - name: Login to Docker Hub
        run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and Push Docker Image
        run: |
          docker build -t your-image-name .
          docker tag your-image-name ${{ secrets.DOCKER_USERNAME }}/your-image-name:latest
          docker push ${{ secrets.DOCKER_USERNAME }}/your-image-name:latest

```

## 8. SSH into VM and Deploy
Add steps to SSH into your VM server and pull the latest Docker image.

```
      - name: SSH into VM and Deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.VM_HOST }}
          username: ${{ secrets.VM_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: ${{ secrets.VM_SSH_PORT }}
          script: |
            docker stop your-container-name || true
            docker rm your-container-name || true
            docker pull ${{ secrets.DOCKER_USERNAME }}/your-image-name:latest
            docker run -d --name your-container-name -p 80:80 ${{ secrets.DOCKER_USERNAME }}/your-image-name:latest

```

## 9. Configure VM Secrets
Add secrets for your VM server details (e.g., VM_HOST, VM_USERNAME, VM_SSH_PORT).

## 10. Commit and Push
Commit the changes to your repository and push them to trigger the GitHub Actions workflow.