# Jenkins & Docker(DIND) Deployment on Kubernetes


# Prerequisite

- Kubernetes Cluster (with default StorageClass) (*Tested on kubernetes 1.26.5)
- Docker
- Operating System (Tested on Ubuntu 22.04)

# Deployment

## Build Image

### Docker (DIND)

DIND(Docker in Docker) is needed to enable docker command that will be use in CI/CD process (Image building, running container, etc)

1. To use specific version of dind, modify Dockerfile.dind file
2. Build docker:dind using docker build
    
    ```bash
    docker build -t ghcr.io/epsindo-ai/docker:jenkins-dind-kubernetes -f Dockerfile.dind .
    ```
    

### Jenkins Server

1. Build the image using Dockerfile provided in this repository
    
    ```docker
    FROM jenkins/jenkins:2.452.3-jdk17
    USER root
    RUN apt-get update && apt-get install -y lsb-release
    RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
      https://download.docker.com/linux/debian/gpg
    RUN echo "deb [arch=$(dpkg --print-architecture) \
      signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
      https://download.docker.com/linux/debian \
      $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
    RUN apt-get update && apt-get install -y docker-ce-cli
    USER jenkins
    RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
    ```
    
2. Build jenkins-server image using docker build command:
    
    ```docker
    docker build -t ghcr.io/epsindo-ai/jenkins:2.452.3-jdk17-docker .
    ```
    

## Kubernetes Deployment

Create a namespace named `devops-tools` using the following command:

```bash
kubectl create namespace devops-tools
```

### Volumes

Enter the following command to create volume

```bash
kubectl apply -f jenkins-docker-volumes.yaml
```

### Jenkins-Docker Deployment & Service (DIND)

Enter the following command to deploy DIND for jenkins

```bash
kubectl apply -f jenkins-docker.yaml
kubectl apply -f jenkins-docker-service.yaml
```

### Jenkins Server Deployment & Service

Enter the following command to deploy jenkins server

```bash
kubectl apply -f jenkins-server-deployment.yaml
kubectl apply -f jenkins-server-service.yaml
```

### Jenkins Ingress(Optional)

Enter the following command to create jenkins ingress

```bash
kubectl apply -f jenkins-ingress.yaml
```

## Verify Deployment

Access the jenkins dashboard through service NodePort/Ingress. If you’re using NodePort access the jenkins dashboard through `http://<host-ip>:<node-port>` you will be required to enter initialAdminPassword, you can retrieve the password by entering the following command:

```bash
kubectl exec -n devops-tools <pods-name> -- cat /var/jenkins_home/secrets/initialAdminPassword
```