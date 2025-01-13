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

# Additional Infrastructure Implementation

## Ansible and Vagrant Integration

### Role Structure and Organization
This deployment now includes Ansible automation with the following structure:
```
ansible-playbook/
├── inventory.yml                   # Inventory configuration
├── group_vars/                     # Group variables
│   └── all.yml                    # Variables for all groups
├── roles/
│   ├── setup-mongodb/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── defaults/              
│   │   │   └── main.yml
│   │   ├── handlers/             
│   │   │   └── main.yml
│   │   └── vars/
│   │       └── main.yml
│   ├── backend-deployment/
│   │   └── [similar structure]
│   └── frontend-deployment/
│       └── [similar structure]
├── playbook.yml
└── ansible.cfg
```

### Virtual Machine Configuration
Enhanced Vagrant setup includes:
```ruby
config.vm.provider "virtualbox" do |vb|
  vb.memory = "2048"
  vb.cpus = 2
end

# Port forwarding
config.vm.network "forwarded_port", guest: 3000, host: 3000  # Frontend
config.vm.network "forwarded_port", guest: 5000, host: 5000  # Backend
config.vm.network "forwarded_port", guest: 27017, host: 27017  # MongoDB
```

### Ansible Playbook Implementation
The playbook executes in the following order:
1. Pre-tasks:
   - System updates
   - Docker installation
   - Network setup
2. Role execution:
   - MongoDB setup
   - Backend deployment
   - Frontend deployment
3. Post-tasks:
   - Health checks
   - Service verification

### Additional Configuration Files

#### Inventory Configuration
```yaml
all:
  hosts:
    myserver:
      ansible_host: 127.0.0.1
      ansible_port: 2222
      ansible_user: vagrant
      ansible_private_key_file: .vagrant/machines/default/virtualbox/private_key
      ansible_python_interpreter: /usr/bin/python3
```

#### Ansible Configuration
```ini
[defaults]
inventory = hosts
remote_user = vagrant
private_key_file = .vagrant/machines/default/virtualbox/private_key
host_key_checking = False
roles_path = roles
retry_files_enabled = False
interpreter_python = auto_silent
```

### Alternative Deployment Method
In addition to the Docker Compose method described above, you can now deploy using Ansible:

1. Start the environment:
```bash
vagrant up
```

2. Run the Ansible playbook:
```bash
ansible-playbook playbook.yml
```

3. Verify deployment:
```bash
curl http://localhost:3000  # Check frontend
curl http://localhost:5000  # Check backend API
```

### Additional Debugging Tips

For Ansible-specific issues:
- Check playbook syntax: `ansible-playbook --syntax-check playbook.yml`
- Run with verbosity: `ansible-playbook -vv playbook.yml`
- Check specific roles: `ansible-playbook playbook.yml --tags "frontend"`

### Enhanced Security Features
The Ansible implementation adds:
- Automated Docker security configuration
- Consistent environment setup
- SSH key-based authentication
- Proper permission management
- Network isolation between services

This additional infrastructure layer provides:
- Fully automated deployment
- Consistent development environments
- Better security practices
- Simplified debugging
- Enhanced monitoring capabilities

# Order of Execution and Reasoning in Playbook

The order of execution in the playbook ensures that dependencies are correctly established before deploying services. Each role has specific functions, as explained below:

### 1. Pre-tasks
Pre-tasks prepare the system for deployment:
- **System updates**: Uses the `apt` module to update and upgrade the system packages.
- **Docker installation**: Ensures Docker is installed using the `community.docker.docker_image` module.
- **Network setup**: Configures network settings using the `ansible.builtin.command` module to create isolated environments for the services.

### 2. Roles Execution
#### a) `setup-mongodb`
This role sets up the MongoDB database:
- **Tasks**:
  - Install MongoDB Docker container.
  - Create persistent volumes using `community.docker.docker_volume`.
  - Verify the MongoDB service using `command` module.
- **Positioning**: Runs first because it provides the backend's data layer.

#### b) `backend-deployment`
This role deploys the backend service:
- **Tasks**:
  - Build and start the backend container using `community.docker.docker_container`.
  - Set environment variables for production.
- **Positioning**: Runs after MongoDB since the backend requires a database connection.

#### c) `frontend-deployment`
This role deploys the React frontend:
- **Tasks**:
  - Build and start the frontend container.
  - Configure network routing to connect with the backend.
- **Positioning**: Runs last as it depends on both the backend and MongoDB.

### 3. Post-tasks
Post-tasks verify that the deployment succeeded:
- **Health checks**: Use the `uri` module to test service endpoints.
- **Service verification**: Restart services if needed using the `service` module.

This sequential order ensures the proper flow and functionality of the application while leveraging Ansible’s modular and automated features.

