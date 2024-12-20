# YOLO E-commerce Application Deployment Guide

## What We're Using and Why

### Our Container Images
I chose specific versions of container images to make our app run smoothly:

1. **Frontend & Backend**: `node:16-alpine3.16`
   - Why? It's super lightweight (only 55MB!) but still has everything we need
   - Perfect for our React frontend and Node.js backend
   - Alpine Linux base means better security with fewer vulnerabilities

2. **Database**: `mongo:6.0`
   - Using the official MongoDB image
   - Version 6.0 is stable and has all the features we need
   - Widely used and well-tested in production

## How We Built Our Containers

### Frontend Container (React App)
I used a two-stage build to keep our final image small and clean:

```dockerfile
# First stage: Build our React app
FROM node:16-alpine3.16 as builder
WORKDIR /client
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Second stage: Only keep what we need to run the app
FROM node:16-alpine3.16
WORKDIR /client
COPY --from=builder /client/build ./build
COPY --from=builder /client/package*.json ./
ENV NODE_ENV=production
EXPOSE 3000
CMD ["npm", "start"]
```

### Backend Container (Node.js API)
Kept it simple but efficient:

```dockerfile
FROM node:16-alpine3.16
WORKDIR /backend
COPY package*.json ./
RUN npm install --production
COPY . .
ENV NODE_ENV=production
EXPOSE 5000
CMD ["npm", "start"]
```

## How Everything Connects

### Network Setup
Set up a private network for our containers to talk to each other:
- Frontend runs on port 3000 (what users see)
- Backend API on port 5000 (handles business logic)
- MongoDB on port 27017 (stores our data)

```yaml
services:
  frontend:
    ports: ["3000:3000"]
  backend:
    ports: ["5000:5000"]
  mongodb:
    ports: ["27017:27017"]
networks:
  app-network:
    driver: bridge
```

### Data Storage
Made sure our MongoDB data sticks around even if containers restart:

```yaml
volumes:
  mongodb_data:
    driver: local
services:
  mongodb:
    volumes:
      - mongodb_data:/data/db
```

## How We Built and Deployed Everything

1. **Getting Started**
   ```bash
   # Clone the project
   git clone git@github.com:Kevin98-x/yolo.git
   cd yolo
   ```

2. **Setting Up Version Control**
   - Created `.gitignore` for node_modules and environment files
   - Set up Docker-related files in each service directory

3. **Building Everything**
   ```bash
   # Build and start everything
   docker-compose build
   docker-compose up
   ```

4. **Pushing to Docker Hub**
   ```bash
   # Tag and push our images
   docker-compose push
   ```

## Deployment Steps:
1. Make sure Docker and Docker Compose are installed
2. Clone the repository
3. Run `docker-compose up`
4. Check everything's running:
   - Frontend: http://localhost:3000
   - Backend: http://localhost:5000
   - MongoDB: localhost:27017

## Debugging Tips
- Check container logs: `docker-compose logs`
- Enter a container: `docker-compose exec service-name sh`
- Restart a service: `docker-compose restart service-name`

This setup makes our application:
- Easy to deploy (just one command!)
- Scalable (can add more containers easily)
- Maintainable (each part is separate but connected)
- Reliable (data persists between restarts)