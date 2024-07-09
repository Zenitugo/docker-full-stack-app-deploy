FROM node:22.4.0-alpine


# Set the working directory inside the container
WORKDIR /app

# Copy the package.json and package-lock.json files to the working directory
COPY package*.json ./

# Install dependencies
RUN npm install

# Set the environmental variable
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL

# Copy the rest of the application files to the working directory
COPY . .

# Expose the port the app runs on
EXPOSE 5173

# Start the development server
CMD ["npm", "run", "dev"]
