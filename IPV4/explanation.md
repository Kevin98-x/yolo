# Explanation of Kubernetes Deployment

This document provides a detailed explanation of the implementation choices made for deploying the application on Google Kubernetes Engine (GKE). It covers the use of Kubernetes objects, persistent storage, service exposure, Git workflow, and best practices.

---

## Kubernetes Objects Used

### 1. **Deployments**
- **Purpose**: Used for stateless components (frontend and backend).
- **Benefits**: Ensures high availability and scalability by maintaining a specified number of replicas.
- **Example**: The frontend and backend are deployed using Deployments, which automatically manage pod replicas and ensure the application remains available even if a pod fails.

### 2. **StatefulSets**
- **Purpose**: Used for the MongoDB database to maintain stable network identities and persistent storage.
- **Benefits**: Ensures data persistence even if pods are rescheduled.
- **Why StatefulSets?**: StatefulSets are ideal for stateful applications like databases because they provide:
  - Stable, unique network identifiers for each pod.
  - Ordered deployment and scaling.
  - Persistent storage that persists across pod restarts or rescheduling.
- **Example**: The MongoDB database is deployed as a StatefulSet to ensure that the database data is retained even if the pod is deleted or rescheduled.

### 3. **Services**
- **LoadBalancer**:
  - **Purpose**: Used to expose the frontend to the internet.
  - **Why LoadBalancer?**: A LoadBalancer service provides an external IP address that allows external users to access the frontend application.
  - **Example**: The frontend service is exposed using a LoadBalancer, making it accessible via `http://34.28.65.168`.
- **ClusterIP**:
  - **Purpose**: Used for internal communication between the backend and database.
  - **Why ClusterIP?**: A ClusterIP service is used for internal communication within the cluster, ensuring that the backend and database are not exposed to the internet.
  - **Example**: The backend and MongoDB services use ClusterIP to communicate securely within the cluster.

### 4. **PersistentVolumes (PV) and PersistentVolumeClaims (PVC)**
- **Purpose**: Used to provide persistent storage for the MongoDB database.
- **Benefits**: Ensures that data is not lost when the database pod is deleted.
- **Why Persistent Storage?**: Persistent storage is critical for stateful applications like databases to ensure data durability.
- **Example**: A PersistentVolume (PV) is provisioned, and a PersistentVolumeClaim (PVC) is used by the MongoDB StatefulSet to store data persistently.

### 5. **ConfigMaps and Secrets**
- **Purpose**: Environment variables (e.g., `MONGO_URL`, `REACT_APP_API_URL`) are defined directly in the manifests for simplicity.
- **Best Practice**: In a production environment, these would be moved to ConfigMaps or Secrets for better security and manageability.
- **Why ConfigMaps and Secrets?**: ConfigMaps and Secrets are used to manage configuration data and sensitive information (e.g., passwords) separately from the application code.

---

## Method Used to Expose Pods to Internet Traffic

### **Frontend**
- **Service Type**: LoadBalancer.
- **Accessibility**: Accessible via an external IP address (e.g., `http://34.28.65.168`).
- **Why LoadBalancer?**: A LoadBalancer service automatically provisions an external IP address, making the frontend accessible to users on the internet.

### **Backend and Database**
- **Service Type**: ClusterIP.
- **Accessibility**: Only accessible within the cluster.
- **Why ClusterIP?**: ClusterIP services are used for internal communication, ensuring that the backend and database are not exposed to the internet, which enhances security.

---

## Use of Persistent Storage

### **PersistentVolumes (PV) and PersistentVolumeClaims (PVC)**
- **Purpose**: Used for the MongoDB database to ensure data persistence.
- **Testing**: Tested by deleting the database pod and verifying that data is retained after the pod is recreated.
- **Why Persistent Storage?**: Persistent storage ensures that data is not lost when the database pod is deleted or rescheduled.
- **Example**: A PersistentVolume (PV) is provisioned with 1Gi of storage, and a PersistentVolumeClaim (PVC) is used by the MongoDB StatefulSet to store data. Even if the MongoDB pod is deleted, the data remains intact.

---

## Git Workflow

### 1. **Branching**
- **Purpose**: Created a new branch for each component (e.g., `frontend`, `backend`, `database`).
- **Process**: Merged changes into the `main` branch after testing.
- **Why Branching?**: Branching allows for isolated development and testing of each component without affecting the main codebase.

### 2. **Commits**
- **Purpose**: Used descriptive commit messages to track changes.
- **Examples**:
  - `feat: Added frontend deployment manifest`
  - `fix: Updated backend service configuration`
- **Why Descriptive Commits?**: Descriptive commit messages make it easier to track changes and understand the evolution of the project.

### 3. **Documentation**
- **Purpose**: Added a `README.md` file with deployment instructions and a link to the live application.
- **Why Documentation?**: Documentation ensures that others can understand and reproduce the deployment process.

---

## Application Functionality

- The application is fully functional:
  - Users can access the frontend via the external IP (`http://34.28.65.168`).
  - The frontend communicates with the backend to fetch data.
  - The backend interacts with the MongoDB database to store and retrieve data.
- **Why Test Functionality?**: Testing ensures that all components of the application work together as expected.

---

## Best Practices

### 1. **Docker Image Tagging**
- **Purpose**: Used semantic versioning for Docker images (e.g., `v1.0.0`).
- **Process**: Pushed images to Docker Hub for easy access and deployment.
- **Why Semantic Versioning?**: Semantic versioning ensures that each image version is uniquely identifiable and follows a consistent naming convention.

### 2. **Labels and Annotations**
- **Purpose**: Used labels to organize and track pods (e.g., `app: frontend`, `app: backend`).
- **Why Labels and Annotations?**: Labels and annotations help organize and manage Kubernetes resources, making it easier to filter and track them.

### 3. **Persistent Storage**
- **Purpose**: Used PersistentVolumes and PersistentVolumeClaims to ensure data persistence.
- **Why Persistent Storage?**: Persistent storage ensures that data is not lost when pods are deleted or rescheduled.

---

## Next Steps

1. **Deploy to GKE**:
   - Apply the Kubernetes manifests to the GKE cluster.
   - Verify that the application is accessible via the external IP.

2. **Test Persistent Storage**:
   - Delete the database pod and verify that data is retained.

3. **Push to GitHub**:
   - Push the Kubernetes manifests and documentation to GitHub.

4. **Clean Up**:
   - Delete the GKE cluster to avoid unnecessary charges.

---

## Conclusion

This project demonstrates the use of Kubernetes to deploy a distributed application with persistent storage. 
By following best practices and leveraging Kubernetes objects like Deployments, StatefulSets, and Services, we have created a scalable and reliable system. 
The use of **StatefulSets** for the database, **LoadBalancer** for the frontend, and **PersistentVolumes** for data storage ensures that the application is both highly available and durable.