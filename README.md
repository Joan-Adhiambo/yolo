# Yolo - E-Commerce Backend with Node.js, Express, and MongoDB

This repository contains a backend application configured to run with Docker and orchestrated using Docker Compose. This README breaks down the steps needed to configure Dockerfiles, run containers, and manage them using Docker Compose.

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
|
├── docker-compose.yaml
|-- explanation.md         
└── README.md             

# Installation
# prerequisites
Node.js(v14 or higher)
MongoDB
Docker, for containerization

# Docker Setup
1. Dockerfile Breakdown

The Dockerfile defines how to build the container

# Backend

# Use official Node.js  image
FROM node:14 AS build

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy source code
COPY . .

# Expose port
EXPOSE 5000

# Start the application
CMD ["npm", "start"]



# frontend
# Use an official Node runtime as a parent image
FROM node:14-alpine AS build

# Set the working directory in the container
WORKDIR /app

# Copy the package.json and package-lock.json files to the container
COPY package*.json ./

# Install application dependencies
RUN npm install

# Copy the rest of the application code to the container
COPY . .

# Build the app from production
RUN npm run build

# Use nginx to serve the build app
FROM nginx:alpine

# Copy the built app from the node build stage
COPY --from=build /app/build /usr/share/nginx/html

# Copy custom nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose the port the app runs on
EXPOSE 3000

# Define the command to run your app
CMD ["nginx","-g", "daemon off;"]

# Key Points:

- Use the Node.js official image for consistency
- Set a working directory for the app inside the container.
- Install dependencies before copying the full source code to leverage Docker layer caching.
- Expose the port the app listens on.
- CMD defines the command to start the container.

2. MongoDB Dockerfile (Optional)

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
   driver: bridge
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

