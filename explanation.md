# Vagrant Implementation
## Overview

This section explains the Vagrant setup used to provision a virtual environment for the Ansible automation in the Configuration Management IP project. 
The purpose is to create an isolated Ubuntu-based virtual machine that acts as the target node for configuration and deployment using Ansible.

# Vagrant Configuration
The base box used in this setup is:-
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

- Base Box:-
Defines the OS image (ubuntu/jammy64).

- Network Configuration:-
The forwarded port allows the web application to be accessed on localhost

- Provisioner: -
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

- The web application can be accessed via the browser at:
http://localhost:5000 //backend
http://localhost:3000  //frontend

# The VM can be accessed  using:
vagrant ssh

# Linking Vagrant to Ansible

- The Vagrantfile is configured to trigger Ansible automatically as part of the provisioning process. 
- This ensures that once the virtual machine is up, Ansible takes over to configure the system using predefined roles and tasks.
- Each role (system-setup, clone-repo, deploy) handles a specific part of the automation:

# system-setup  Role
- Sets up the required tools like Docker, Git, and Docker-compose

# clone-repo Role
- pulls the web application code from the GitHub repository:
  https://github.com/Joan-Adhiambo/yolo.git

# deploy Role
- sets up and starts the application inside Docker containers.

This modularization allows for a clean and configuration that is easy to manage.

# Conclusion
- The Vagrant configuration ensures an automated and portable development environment. 
- It serves as the foundation for Ansible to perform further configuration tasks such as dependency installation, container setup, and application        deployment without manual intervention.






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
I used mvertes/alpine-mongo:latest

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



