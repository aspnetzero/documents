# Publishing to Docker Containers

ASP.NET Zero solution has a **build folder** which contains scripts to **build & publish** your solution to the **output** folder.

## Prerequisites

* Node.js (LTS version recommended)
* Docker Desktop installed and running
* Packages installed (`yarn` or `npm install`)

## Building the React Application

Before creating a Docker image, build the React application:

```bash
yarn build
# or
npm run build
```

This will create a production build in the `dist/` folder using Vite.

## Creating a Docker Image

### Sample Dockerfile

Create a `Dockerfile` in the project root:

```dockerfile
# Build stage
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Sample nginx.conf

Create an `nginx.conf` file for SPA routing:

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### Building and Running

Build the Docker image:

```bash
docker build -t aspnetzero-react .
```

Run the container:

```bash
docker run -d -p 4200:80 aspnetzero-react
```

## Docker Compose

For running with Docker Compose, create a `docker-compose.yml`:

```yaml
version: '3.8'
services:
  react-app:
    build: .
    ports:
      - "4200:80"
    environment:
      - NODE_ENV=production
```

Run with:

```bash
docker compose up -d
```

## Configuration

Before building the Docker image, ensure your API endpoint is correctly configured. Update the remote service URL in your environment configuration to point to your production API server.

## Notes

* The React UI is a static SPA that connects to the ASP.NET Core backend API
* Ensure CORS is properly configured on your backend to allow requests from the React container
* For production deployments, consider using HTTPS with proper SSL certificates 


