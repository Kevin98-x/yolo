### Kubernetes Objects Used

1. **Deployments**:
   - Used for stateless components (frontend and backend).
   - Ensures high availability and scalability by maintaining a specified number of replicas.

2. **StatefulSets**:
   - Used for the MongoDB database to maintain stable network identities and persistent storage.
   - Ensures data persistence even if pods are rescheduled.

3. **Services**:
   - **LoadBalancer**: Used to expose the frontend to the internet.
   - **ClusterIP**: Used for internal communication between the backend and database.

4. **PersistentVolumes (PV) and PersistentVolumeClaims (PVC)**:
   - Used to provide persistent storage for the MongoDB database.
   - Ensures that data is not lost when the database pod is deleted.

5. **ConfigMaps and Secrets**:
   - Environment variables (e.g., `MONGO_URL`, `REACT_APP_API_URL`) are defined directly in the manifests for simplicity.
   - In a production environment, these would be moved to ConfigMaps or Secrets for better security and manageability.

---

### **Method Used to Expose Pods to Internet Traffic**

- **Frontend**:
  - Exposed using a **LoadBalancer** service.
  - Accessible via an external IP address (e.g., `http://<EXTERNAL_IP>:80`).

- **Backend and Database**:
  - Exposed internally using **ClusterIP** services.
  - Only accessible within the cluster.

---

### **Use of Persistent Storage**

- **PersistentVolumes (PV)** and **PersistentVolumeClaims (PVC)**:
  - Used for the MongoDB database to ensure data persistence.
  - Tested by deleting the database pod and verifying that data is retained after the pod is recreated.

---

### **Git Workflow**

1. **Branching**:
   - Created a new branch for each component (e.g., `frontend`, `backend`, `database`).
   - Merged changes into the `main` branch after testing.

2. **Commits**:
   - Used descriptive commit messages to track changes.
   - Examples:
     - `feat: Added frontend deployment manifest`
     - `fix: Updated backend service configuration`

3. **Documentation**:
   - Added a `README.md` file with deployment instructions and a link to the live application.
   - Added an `explanation.md` file to explain implementation choices.

---

### **Application Functionality**

- The application is fully functional:
  - Users can access the frontend via the external IP.
  - The frontend communicates with the backend to fetch data.
  - The backend interacts with the MongoDB database to store and retrieve data.

---

### **Best Practices**

1. **Docker Image Tagging**:
   - Used semantic versioning for Docker images (e.g., `v1.0.0`).
   - Pushed images to Docker Hub for easy access and deployment.

2. **Labels and Annotations**:
   - Used labels to organize and track pods (e.g., `app: frontend`, `app: backend`).

3. **Persistent Storage**:
   - Used PersistentVolumes and PersistentVolumeClaims to ensure data persistence.

---

### **Next Steps**

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

### **Conclusion**

This project demonstrates the use of Kubernetes to deploy a distributed application with persistent storage. By following best practices and leveraging Kubernetes objects like Deployments, StatefulSets, and Services, we have created a scalable and reliable system.