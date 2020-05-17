# About the Project - Udagram Microservices
Udagram is a simple cloud application developed alongside the Udacity Cloud Engineering Nanodegree. It allows users to register and log into a web client, post photos to the feed, and process photos using an image filtering microservice. Following are the services involved in this project:

Correspondingly, the project is split into following parts:
1. The RestAPI Feed Backend, a Node-Express feed microservice.
1. The RestAPI User Backend, a Node-Express user microservice.
1. The Simple Frontend - A basic Ionic client web application which consumes the RestAPI Backend.
1. Nginx as a reverse-proxy server, when different backend services are running on the same port, then a reverse proxy server directs client requests to the appropriate backend server and retrieves resources on behalf of the client.  

## Dependencies and tools

Tools: npm, Node, ionic cli, Docker, Kubernetes, kubectl, Dockerhub account, travis
AWS: aws account, s3 bucket, aws rds postgress database, aws eks cluster

## Config

1. Add file `configmap` with example content

```bash
    AWS_BUCKET=your_aws_bucket_name
    AWS_PROFILE=your_aws_app_profile
    AWS_REGION=your_aws_region
    JWT_SECRET=your_jwt_secret
    POSTGRESS_DATABASE=your_db_name
    POSTGRESS_HOST=your_db_host
    POSTGRESS_DIALECT=postgres
    URL=http://localhost:8100 
```

2. Add file `dbusername.txt` with content

```bash
    your_user_name
```
3. Add file `dbuserpassword.txt` with content

```bash
    your_user_password
```
4. Add file `credentials` - file with your aws credentials

## Run application

### 1. Run with Docker

Build docker container
```bash
docker-compose -f deployment/docker/docker-compose-build.yaml build --parallel
```
Run docker container
```bash
docker-compose -f deployment/docker/docker-compose.yaml up
```

App is running at http://localhost:8100

### 2. Run with Kubernetes

Create database credentials (env-secret):
```bash
kubectl create secret generic env-secret --from-file=POSTGRESS_USERNAME=./config/dbusername.txt --from-file=POSTGRESS_PASSWORD=./config/dbuserpassword.txt
```

Create aws credentials (aws-secret):
```bash
kubectl create secret generic aws-secret --from-file=./config/credentials
```

Create configmap:
```bash
kubectl create configmap env-configmap --from-env-file=./config/configmap
```

Push all docker images to public Dockerhub account

Create Kubernetes deployments:

```bash
kubectl apply -f deployment/k8s/backend-feed-deployment.yaml
kubectl apply -f deployment/k8s/backend-feed-service.yaml
kubectl apply -f deployment/k8s/backend-user-deployment.yaml
kubectl apply -f deployment/k8s/backend-user-service.yaml
kubectl apply -f deployment/k8s/frontend-deployment.yaml
kubectl apply -f deployment/k8s/frontend-service.yaml
kubectl apply -f deployment/k8s/reverseproxy-deployment.yaml
kubectl apply -f deployment/k8s/reverseproxy-service.yaml
```

Check results:

```bash
kubectl get deployments
kubectl get services
kubectl get pods
kubectl get svc
```

Port forwarding:

```bash
kubectl port-forward service/frontend 8100:8100
kubectl port-forward service/reverseproxy 8080:8080
```

App is running at http://localhost:8100

### 3. A-B testing

```bash
git checkout app-v2
```

Create docker image:
```bash
docker build -t <image-name>/<image-id> .
```
Push image to Dockerhub:

```bash
docker push <image-name>/<image-id>
```

Create `B` deployment:

```bash
kubectl apply -f deployment/k8s/frontend-deployment.v2.yaml
```

Check results:

```bash
kubectl get deployments
kubectl get pods
```

### 4. Rolling updates

```bash
git checkout rolling-updates
```

```bash
kubectl apply -f deployment/k8s/frontend-deployment.yaml
```

Check results:

```bash
kubectl get deployments
kubectl get pods
kubectl describe deployment udagram-frontend
```
