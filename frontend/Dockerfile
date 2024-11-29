# Use Node.js 20 Alpine base image
FROM node:20-alpine3.18

# Set working directory
WORKDIR /app

# # Install system dependencies required for Node.js
# RUN apk update && apk add --no-cache bash curl

# Copy only necessary files to leverage Docker caching
COPY package.json .

# Configure npm retry settings and install dependencies
RUN npm set fetch-retries 5 && \
    npm set fetch-retry-mintimeout 20000 && \
    npm set fetch-retry-maxtimeout 120000 && \
    npm install && \
    npm install -g pm2

# Copy the rest of the application files
COPY . .

# Install dependencies and pm2
RUN npm install
RUN npm install -g pm2

# Copy and set permissions for start script
COPY start.sh /app/start.sh
RUN chmod +x /app/start.sh

# Expose the application port
EXPOSE 3000

# Start the application
CMD ["/app/start.sh"]


RUN npm install -g pm2