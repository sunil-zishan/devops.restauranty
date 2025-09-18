# Use Node.js base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files and install dependencies
COPY package*.json ./
RUN npm install

# Copy source code
COPY . .

# Expose the service port (set via .env)
EXPOSE 3002

# Start the service with node instead of nodemon
CMD ["node", "server.js"]
