# Use the latest official Node.js image as a base
FROM node:latest

# Set the working directory
WORKDIR /app

# Copy the application files
COPY . .

# arg command
ARG VITE_API_URL=${VITE_API_URL}

# Install dependencies
RUN npm install

# Expose the port the development server runs on
EXPOSE 5173

# Run the development server
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
