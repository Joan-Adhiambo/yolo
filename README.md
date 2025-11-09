
# E-commerce Application Deployment on Kubernetes

This repository contains Kubernetes manifests and configurations for deploying ane-commerce application (frontend, backend, and database) on Google Kubernetes Engine (GKE).

## Live Application

 Access Link:
http://136.119.134.23/  deployed using GKE
http://192.168.49.2:31000/   deployed locally using minikube


# Kubernetes Architecture
### Components
- Frontend	Deployment + Service 
- Backend	Deployment + Service + Ingress	
- Database	StatefulSet + PVC + Service	MongoDB with persistent volume for data retention

- Requests are sent → GKE LoadBalancer (External Ip)
- Backend-ingress controller routes request → Backend service.
- Backend stores/retrieves data from the StatefulSet database.

# Kubernetes Objects Used

- Deployments: frontend and backend services.
- StatefulSet: For database with persistent storage.
- Backend-Services: ClusterIP for internal access, Ingress for public access.
- Frontend-service: Load Balancer which exposes the service externally. The external IP is automatically  provisioned by the GKE 
- Ingress Controller:with defined origin, cors rules. The Ingres Routes external traffic to backend services.
- PersistentVolumeClaim: Ensures database persistence and used Headderless cluster for mongo-service

# Structure
.
├── manifests/
│   ├── frontend-deployment.yaml
│   ├── backend-deployment.yaml
│   ├── mongo-statefulset.yaml
│   ├── backend-services.yaml
|   |-- frontend-service.yaml
│   ├── backend-ingress.yaml
├── Docker-compose
├── explanation.md
└── README.md

# Deployment Steps

Clone this repository

git clone https://github.com/Joan-Adhiambo/yolo.git


Create your namespace (optional)

kubectl create namespace <namespacename>

Create Manifests eg
- kubectl touch backend-deployment.yaml

Apply manifests

- kubectl apply -f manifests/


- Check Pods and Services

    kubectl get pods
    kubectl get svc
    kubectl get ingress
    Kubectl get pvc


# Access Application
- Use the external IP provisoned by GKE

### when deploying locally
- Get your external IP and open in browser:
- minikube service <service name>  

# Troubleshooting
- Check logs of the services to confirm each service is well confgured and also the pods are up and running.
- CORS Errors: Ensure backend has proper CORS configuration or it is enabled.
- Connection Refused: use ingress contoller to route traffic to external service


# Outcome

- Frontend and backend deployed successfully.
- Stateful database retains data after Pod deletion.
- Application accessible via External IP.
































***********************************************************************************************************************************************************************


# Configuration Management I– Ansible Automation

This project implements infrastructure automation using Vagrant and Ansible.
Vagrant provisions a virtual machine running Ubuntu, while Ansible automates configuration and deployment of the application.

The application runs on Node.js, uses MongoDB (hosted in the cloud), and is containerized with Docker for consistency.

# Project Structure
project-root/
├── Vagrantfile
├── playbook.yaml
├── roles/
│   ├── system-setup/
│   │   ├── tasks/main.yml
│   │   |
│   ├── clone-repo/
│   │   ├── tasks/main.yml
|   |   |--- vars/main.yml
│   ├── deploy/
│   │   ├── tasks/main.yml
│
|---> hosts   
├── explanation.md
└── README.md

# Vagrant Setup

The Vagrantfile provisions an Ubuntu VM and runs the playbook.yaml automatically during provisioning.
Ports are forwarded to allow access to the web application on:

http://localhost:5000
http://localhost:3000


- The Virtual machine can be accessed using the command:-

vagrant ssh

- To check the containers running run the command:-
 sudo docker ps


# Ansible Playbook Overview

The playbook.yaml orchestrates the execution of all defined roles in order.
It defines:-

- Variables for configurations and file paths
- Roles(system setup, cloning, deployment)
- Tags

# Ancible structure:

[defaults]
remote_user =vagrant
private_key_file = /home/user/assignment/yolo/.vagrant/machines/default/virtualbox/private_key
invetory = hosts
roles_path = ./roles

# Ansible Roles
## system-setup

- Purpose: Prepares the server with all necessary dependencies and environment setup.
Tasks:

- Update and upgrade Ubuntu packages
- Install Docker, Docker Compose, Git
- Start and enable the Docker service

- Tags: setup, dependencies

task:
- name: Update apt and install Git & Docker
  apt:
    name:
      - git
      - docker.io
      - docker-compose
    state: present
    update_cache: yes

- name: Ensure Docker service is running
  service:
    name: docker
    state: started
    enabled: yes

## clone-repo

- Purpose: Clones the project source code from GitHub into the virtual machine.
- Key Tasks:

- Ensure Git is installed
- Clone the repository from GitHub into /opt/yolo
- Ensures successful cloning

- Tags: clone, repo

task:-
- name: Clone YOLO repository
  git:
    repo: "{{ repo_url }}"
    dest: "{{ app_dir }}"
    force: yes
    

## deploy
- Purpose: Handles running the application using Docker.
- Key Tasks:

- Copy Docker configuration files
- Build Docker images
- Start containers for backend and frontend services
- Ensure application runs on port 5000 and port 3000

- Tags: deploy, docker

task:-
-  name: Run YOLO app with Docker Compose
  command: docker-compose up -d
  args:
    chdir: "{{ app_dir }}"

# Running the Project
Step 1: Start the VM
vagrant up

Step 2: Provision  
vagrant provision

Step 3: Access the App

Once deployment completes successfully:

http://localhost:5000
http://localhost:3000

Step 4: SSH into VM for verification
vagrant ssh

Testing Functionality

Visit the running application in your browser.

Use the “Add Product” form to confirm that the product addition feature works, this validates backend–frontend–database connectivity

**********************************************************************************************************************************************************************
















# Yolo - E-Commerce Backend with Node.js, Express, and MongoDB

This repository contains a backend application configured to run with Docker and orchestrated using Docker Compose. 
This README breaks down the steps taken to configure Dockerfiles, run containers, and manage them using Docker Compose.

# Project Overview

Backend: Node.js + Express

Database: MongoDBAtlas

Containerization: Dockerfile

Orchestration: Docker Compose

# Project folder

yolo/
├── backend/  
|   |-- models
|   |-- routes
│   ├──.env/        
│   ├── Dockerfile/          
│   ├── /package.json        
│   └── server.js   
|
|---client/  
|   |-- public
|   |-- src               
|   ├── Dockerfile          
|   ├── nginx.conf              
|   ├── package.json  
|   |
docker-compose.yaml
---explanation.md         
└── README.md             

# Installation
# prerequisites
Node.js(v14 or higher)
MongoDB
Docker, for containerization

# Docker Setup
1. Dockerfile Breakdown

The Dockerfile defines how to build the container

# 1. Backend Dockerfile

## Use official Node.js  image
FROM node:14 AS build

## Set working directory
WORKDIR /app

## Copy package files
COPY package*.json ./

## Install dependencies
RUN npm install

## Copy source code
COPY . .

## Expose port
EXPOSE 5000

## Start the application
CMD ["npm", "start"]



# 2. frontend Dockerfile

## Use an official Node runtime as a parent image
FROM node:14-alpine AS build

## Set the working directory in the container
WORKDIR /app

## Copy the package.json and package-lock.json files to the container
COPY package*.json ./

## Install application dependencies
RUN npm install

## Copy the rest of the application code to the container
COPY . .

## Build the app from production
RUN npm run build

## Use nginx to serve the build app
FROM nginx:alpine

## Copy the built app from the node build stage
COPY --from=build /app/build /usr/share/nginx/html

## Copy custom nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

## Expose the port the app runs on
EXPOSE 3000

## Define the command to run your app
CMD ["nginx","-g", "daemon off;"]

# Key Points:

- Use the Node.js official image for consistency
- Set a working directory for the app inside the container.
- Install dependencies before copying the full source code to leverage Docker layer caching.
- Expose the port the app listens on.
- CMD defines the command to start the container.

# 3. MongoDB Dockerfile (Optional)

You can either use the official MongoDB image depending with the  custom configurations required and the final image size needed.

image:mvertes/alpine-mongo:latest   
EXPOSE 27017

# Docker Compose Setup

The docker-compose.yml orchestrates multiple containers (backend,database,frontend):


services:
  # Backend
  backend:
    build: ./backend
    image: yolo-backend:v2.0.0
    container_name: backend
    ports:
      - "5000:5000"
    depends_on:
     - app-mongo
    env_file:
      - ./backend/.env
    networks:
      - back-network
      - client-network
  
  # database
  app-mongo:
    image: mvertes/alpine-mongo:latest
    container_name: my-mongo
    ports:
     - "8000:27017"
    networks:
     - back-network

  # Frontend
  client:
    build: ./client
    image: yolo-frontend:v1.0.0
    container_name: client
    ports:
      - "3000:3000"
    depends_on:
      - backend
    networks:
      - client-network
      

networks:
  back-network:
  client-network:
    driver: bridge

# Explanation:

Services: Defines backend and mongo containers.
depends_on: Ensures the database starts before the backend.
Volumes: Persists MongoDB data between container restarts, when necessary
Networks: Allows containers to communicate internally. 

# Running the Containers

Build and start all containers:

docker compose up --build


# Stop and remove containers:

docker compose down


# Check logs for a service:

docker compose logs -f <container name>


Access backend API at:

http://localhost:5000


Access frontend API at:

http://localhost:3000

# Customization Steps

- Change ports if your system uses them, this is to ensure the containers run without conflict.
- Add environment variables in docker-compose.yml or .env for database connection.

# Contributing

Contributions are welcome! Please fork the repository, create a new branch, and submit a pull request.

