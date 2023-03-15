# Reddit Clone App on Kubernetes with Ingress
This project demonstrates how to deploy a Reddit clone app on Kubernetes with Ingress and expose it to the world using Minikube as the cluster.

## AWS Instance details:
OS : Ubuntu 22.04
Servers :
1.CI 
2.Deployment

## Prerequisites
Before you begin, you should have the following tools installed on your local machine: 

- Docker
- Minikube cluster ( Running )
- kubectl
- Git

## Installation of Prerequisites

### Installation in CI Server (Docker Installation)
```
sudo apt-get update -y
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker
```

### Installation in Deployment Server(Minikube & Kubectl Installation)
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
sudo snap install kubectl --classic
minikube start --driver=docker
```

## Step 1: Clone the source code
The first step is to clone the source code for the app. You can do this by using this command git clone 
```
https://github.com/LondheShubham153/reddit-clone-k8s-ingress.git
```

## Step 2: Containerize the Application using Docker
```
FROM node:19-alpine3.15

WORKDIR /reddit-clone

COPY . /reddit-clone

RUN npm install 

EXPOSE 3000

CMD ["npm","run","dev"]
```

## Step 3: Building Docker-Image
Now it's time to build Docker Image from this Dockerfile.Use this command to build a docker image.
```
docker build -t <DockerHub_Username>/<Imagename> .
```

## Step 4: Push the Image To DockerHub
Now push this Docker Image to DockerHub so our Deployment file can pull this image & run the app in Kubernetes pods.
1) First login to your DockerHub account using Command
```
docker login
```
2) Then docker push command for pushing to the DockerHub.
```
docker push <DockerHub_Username>/<Imagename>
```
3) You can use an existing docker image i.e shubhzzz19/reddit-clone for deployment.

## Step 5: Write a Kubernetes Manifest File
When you are going to deploy to Kubernetes or create Kubernetes resources like a pod, replica-set, config map, secret, deployment, etc, you need to write a file called manifest that describes that object and its attributes either in yaml or json. Just like you do in the ansible playbook.

Of course, you can create those objects by using just the command line, but the recommended way is to write a file so that you can version control it and use it in a repeatable way.

1) Write Deployment.yml file: Let's Create a Deployment File For our Application. Use the following code for the Deployment.yml file.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-clone-deployment
  labels:
    app: reddit-clone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: reddit-clone
  template:
    metadata:
      labels:
        app: reddit-clone
    spec:
      containers:
      - name: reddit-clone
        image: trainwithshubham/reddit-clone
        ports:
        - containerPort: 3000
```
2) Write Service.yml file
```
apiVersion: v1
kind: Service
metadata:
  name: reddit-clone-service
  labels:
    app: reddit-clone
spec:
  type: NodePort
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 31000
  selector:
    app: reddit-clone
```
## Step 6: Deploy the app to Kubernetes & Create a Service For It

Now, we have a deployment file for our app and we have a running Kubernetes cluster, we can deploy the app to Kubernetes. To deploy the app you need to run following the command: 
```
kubectl apply -f Deployment.yml
```

Just Like this create a Service using
```
kubectl apply -f Service.yml
```

If You want to check your deployment & Service use the command
```
kubectl get deployment
kubectl get services
```


