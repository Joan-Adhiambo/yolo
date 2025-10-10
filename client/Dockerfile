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

#Build the app from production
RUN npm run build

#Use nginx to serve the build app
FROM nginx:alpine

#Copy the built app from the node build stage
COPY --from=build /app/build /usr/share/nginx/html

#Copy custom nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose the port the app runs on
EXPOSE 3000

# Define the command to run your app
CMD ["nginx","-g", "daemon off;"]