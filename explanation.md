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
- /usr/share/nginx/html: Destination path in the Nginx container. This is the default directory that Nginx uses to serve files.

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



