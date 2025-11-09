# ORCHESTRATION EXPLANATION

# Choice of Kubernetes Objects
- For each object I didn't specify namespace, and by default the services,pods,statefulset were created in the default namespace.

### Database (MongoDB): I used a StatefulSet.
- Reasons:-
- Persistent storage even when pods are deleted. This helps with data consistency and percistence.
- Ordered deployment and scaling, which is important for stateful applications like databases.

### Backend and Frontend: I used Deployments to ensure:

- Easy scaling via replicas.
- Rolling updates without downtime.
- Automatic self-healing of pods if any crashes occur.
- For backend-service I used clusterIp type so that frontend service could use the virtual IP (cluster IP) to access backend pods
- Frontend-service  I used LoadBalancer whcich forwards traffic and distrubtes traffic accross the pods.

### Labels and Annotations:
- All pods include labels (app) to track frontend, backend, and database pods
- This makes management and service selection easier.

# Method Used to Expose Pods

### Frontend: Exposed to the internet using a LoadBalancer Service.

- Automatically provisions a public IP address for the frontend.
- Allows users to access the application directly via the external IP.

### Backend: Exposed internally using a ClusterIP Service
- Allowing the frontend pods to communicate with the ackend service without exposing it to the internet.
- For successful connection to the Database I ensured the the database configurations in .env and also specified in backend-deployment.yaml.

### Use of Ingress Controller
- I also used backend-ingress controller configured rules, the ingress routed traffic to my backend-service on port 5000.
- This ensured the backend service is reachabe without exposng it externally.
### Database: Exposed internally via a headless ClusterIP Service.
- For stable networking between pods.

# Use of Persistent storage
- For the database (MongoDB) I created a PersistentVolumeClaim (PVC) and bound it to a PersistentVolume (PV).
- This is to ensure that data is not lost when Pods are deleted or restarted
- Scaling the StatefulSet doesn’t affect existing data.
- To confirm data persistence I deleted the database pods and confirmed the and the added products remained.


![

](<Screenshot from 2025-11-09 23-25-05.png>)


# Git Workflow

- confirmed the app is running locally
- Taggedg and pushed images to Docker Hub with proper tags (username/app-name:v1).
- Kubernetes manifests creation.
  manifests:-
      - backend-deloyment.yaml
      - backend-service.yaml
      - frontend-deployment.yaml
      - frontend-service.yaml
      - mongo-statefulset.yaml
      - backend-ingress.yaml
- Added persistent volume setup.
- Backend ingress controller configuration and deployment testing.
- Updated documentation (README.md and explanation.md).
- Used descriptive commit messages to track progress clearly.

# Debbungging and Testing
- During deployment, I encountered issues such as:
- CORS errors between frontend and backend (resolved using backend-ingress and enabling cors to be true in rules).
- Connection refused errors, this one also was resolved by configuring backend-ingress to allow  routing of traffic to backend without exposing it externally.
- After debugging, the was successfully accessible and was able to add products and confirm data persistence.

# Docker Image Tagging and Naming
I followed the best practice of tagging Docker images with both version and docker hub name.
![
  
](<Screenshot from 2025-11-09 23-41-22.png>)
![alt text](<Screenshot from 2025-11-09 23-42-07.png>)



















***********************************************************************************************************************************************************************


# Vagrant Implementation
## Overview

This section explains the Vagrant setup used to provision a virtual environment for the Ansible automation in the Configuration Management IP project. 
The purpose is to create an isolated Ubuntu-based virtual machine that acts as the target node for configuration and deployment using Ansible.

# Vagrant Configuration
I used base box used in this setup is:-
ubuntu/jammy64

## Reasons
This is an official Ubuntu box maintained by HashiCorp, ensuring stability and reliability. 
It provides a clean environment for installing dependencies and running the application.


# Vagrantfile Structure
Vagrant.configure("2") do |config|
  
  # Specify the base box
  config.vm.box = "ubuntu/jammy64"
  
  # Define network settings
  config.vm.network "forwarded_port", guest: 5000, host: 5000
  config.vm.network "forwarded_port", guest: 3000, host: 3000
  
  # Provisioning using Ansible
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yaml"
    ansible.verbose = "v"
  end
end

# Explanation

- specify the base box:-
Defines the OS image (ubuntu/jammy64).

- Define Network Settings:-
The forwarded port allows the web application to be accessed on localhost

- Provisioning using Ansible:-
Uses Ansible to automatically configure the VM by running the playbook.yaml file after provisioning.

# Provisioning Process

Run the command:-
vagrant provision

# Vagrant:

- Downloads and initializes the Ubuntu box (if not already present).
- Starts the virtual machine.
- Automatically runs the Ansible playbook (playbook.yaml) defined in the provisioning section.
- Sets up the required environment  ie installing Docker, cloning the GitHub repository, configuring roles
- If any updates are made to the playbook or roles, the provisioning process can be re-run using:

  vagrant provision --reload

# Accessing the Application

- Once provisioning is successful:

- I was able to access the application on brower and add products.
http://localhost:5000 //backend
http://localhost:3000  //frontend

# Image
![alt text](<Screenshot from 2025-10-27 20-51-24.png>)

# The VM can be accessed  using:
vagrant ssh

# Linking Vagrant to Ansible

- Configured Vagrantfile to trigger Ansible automatically as part of the provisioning process. 
- This ensures that once the virtual machine is up, Ansible takes over to configure the system using predefined roles and tasks.
- The roles defined (system-setup, clone-repo, deploy) each handles a specific part of the automation:

# system-setup  Role
- Sets up the environment for Virtual machine

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

# Explanation
- Sets up the required tools like Docker, Git, and Docker-compose
- Docker allows the application to run inside containers.
- Docker Compose manages the container application.
- Git fetches source code from version control repositories (Git hub)

Key Tasks:
- Update and install Ubuntu system packages.
- Install Docker, Docker Compose, and Git.
- Ensure the Docker service is running and enabled on startup.

# clone-repo Role
- clones the project source code from a Git repository into the virtual machine.

- name: Clone YOLO repository
  git:
    repo: "{{ repo_url }}"
    dest: "{{ app_dir }}"
    force: yes

- Variables (configured variables):-
 vars:
  repo_url: "https://github.com/Joan-Adhiambo/yolo.git"  # GitHub repo to clone
  app_dir: "app/yolo"                                   # app directory in VM


Key tasks:
- Ensure Git is installed
- Clone the repository from the  GitHub URL into the /opt/yolo directory.
- Verify that the repository was cloned successfully.

# deploy Role
- Deploys the application using Docker

- name: Run YOLO app with Docker Compose
  command: docker-compose up -d
  args:
    chdir: "{{ app_dir }}"

Key tasks:
- Copy or verify that Dockerfile, docker-compose.yaml are in place.
- Build Docker images as in the configuration.
- Run Docker Compose to start all containers and since the application is containerized there was no need of installing applications dependencies.
- Verify the application is up and running.



# Conclusion
- This modularization allows for a clean and configuration that is easy to manage.
- The Vagrant configuration ensures an automated and portable development environment also enables Ansible to perform further configuration tasks such as dependency  installation, container setup, and application ,deploying without manual intervention.













***********************************************************************************************************************************************************



# Choice of Image

Each container, I carefully selected the base image to optimize performance and image size:

## Backend
I used node:14-alpine AS build

Reasons: 
The Alpine variant is lightweight, reducing the image size, and Node.js 14 is stable and compatible with the application dependencies.

## Frontend
I used node:14-alpine AS build on the first stage
Used nginx:alpine on the second stage

Reasons:
Node 14-alpine is a lightweight Node.js image based on Alpine Linux, which keeps the final image smaller.
It is a stable version for most React or Node.js applications.
Using AS build sets up a multi-stage build, this helped to reduces the final image size.

## Database (MongoDB)
I used mongo:6.0

Reasons:
It is suitable for test and suitable for final smaller images due to it's lightweight.
Improves build times and reduce resource usage.

# Dockerfile Directive
- Explains key lines in a Dockerfile that builds  the containers for each micro-service.

## backend container

FROM node:14-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["node", "server.js"]

- FROM node:14-alpine: Uses a lightweight Node.js runtime that reduces image size.
- WORKDIR /app: Sets a consistent working directory inside the container.
- COPY package*.json ./: Copies only dependency files first.
- RUN npm install: Installs all Node.js dependencies during build time.
- COPY . .: copies  application files after dependencies are installed.
- EXPOSE 5000: The port the app listens on.
- CMD ["node", "server.js"]: Runs the backend server when the container starts.

## frontend container

## Stage 1: Build
FROM node:14-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

## Stage 2
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 3000
CMD ["nginx", "-g", "daemon off;"]

Multi-stage build:
- Stage 1 (Node.js): Build the app, its dependencies and build tools
- Stage 2 (Nginx): Serves the static build files, producing a smaller image.

## COPY --from=build /app/build /usr/share/nginx/html: 
- This line copies the application built from the first build stage (the Node.js stage) into the Nginx image

## Breakdown
- from=build: Transfers build files from the first stage defined as FROM node:14-alpine AS build to Nginxcontainer.
- /app/build: Source path inside the Node.js container.
- /usr/share/nginx/html: Destination path in the Nginx container. This is the default directory that Nginx use to serve files.

## COPY nginx.conf /etc/nginx/conf.d/default.conf: 
- Replaces the default Nginx configuration file with a custom one, nginx.conf.

## Breakdown
- nginx.conf: Your custom Nginx configuration file, stored locally.
- /etc/nginx/conf.d/default.conf: The location of Nginx's default site configuration file inside the container.

- EXPOSE 3000: port for serving the frontend.
- CMD ["nginx", "-g", "daemon off;"]: Starts Nginx in the foreground to keep the container running.

## Database container (MongoDB)

- Used mvertes/alpine-mongo:latest to build the container image.
- No Dockerfile created because the image pulled already includes all necessary configuration for running MongoDB securely.
- Environment Variables: Used .env file in backend to the set  username and password for DB connection.
- EXPOSE 27017: Documents MongoDB port for internal networking.


# Docker-compose networking.
networks:
  back-network:
    driver: bridge
  client-network:
    driver: bridge

- The two networks were defined, that is back-network and client-network including a bridge driver.
- This is to ensure communication between containers.
- The backend container communicates with database via back-network and frontend container via client-network
- Frontend communites with client-network only.

# Docker compose volume
- Explanation.
- volumes: since the DB is cloud hosted, it was not necessary to persist data locally, this is because data is already persisted remotely.

# Git Workflow
  ## Branching
 - master branch

## Descriptive Commits
- Add Dockerfile for backend container.
- Add Dockerfile for frontend container.
- Add .env file in backend.
- Add custom nginx.conf in client, this is to ensure routing works and static files are served (Redirecting all requests to index.html )
- Configure Docker Compose file
- Tagging of images
- Docker image tags aligned with Git tags: v1.0.0, v2.0.0.
    * yolo-frontend   v1.0.0     54.8MB
    * yolo-backend    v2.0.0     147MB
    * mvertes/alpine-mongo       123MB
  


# Images pushed on Dockerhub

![alt text](<Screenshot from 2025-10-11 00-24-41-2.png>)






