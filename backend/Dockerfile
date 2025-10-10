# Stage 1: Build the Node.js app
FROM node:14-alpine AS build

WORKDIR /app

# Copy package files
COPY package*.json ./

#Install  dependecies
RUN npm install

# Copy the application
COPY . .

# Run the app stage
FROM node:14-alpine

WORKDIR /app

# Copy built files and dependencies from the build stage
COPY --from=build /app /app

# Expose the app port
EXPOSE 5000

# Start the app
CMD ["node", "server.js"]