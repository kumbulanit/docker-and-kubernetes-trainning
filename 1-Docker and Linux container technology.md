H

---

### 1. **Introduction to Docker and Containers**
   **Objective:** Understand the fundamentals of Docker and containers, learn how to install Docker, and run a basic container.
   - **Exercise:** 
     1. Install Docker on your Linux machine.
     2. Verify the installation with `docker --version`.
     3. Pull the `nginx` image: 
        ```bash
        docker pull nginx
        ```
     4. Run a container:
        ```bash
        docker run -d -p 8080:80 nginx
        ```
     5. Open a browser and go to `http://localhost:8080` to see the default `nginx` welcome page.

---

### 2. **Docker Images and Containers**
   **Objective:** Learn the difference between images and containers, create and manage containers, and explore container states.
   - **Exercise:**
     1. List downloaded images:
        ```bash
        docker images
        ```
     2. Run a new `nginx` container:
        ```bash
        docker run --name web-container -d -p 8081:80 nginx
        ```
     3. List running containers:
        ```bash
        docker ps
        ```
     4. Stop and start the `web-container`:
        ```bash
        docker stop web-container
        docker start web-container
        ```

---

### 3. **Dockerfile Basics and Custom Images**
   **Objective:** Create a custom Docker image using a Dockerfile and understand basic instructions.
   - **Exercise:**
     1. Create a directory named `custom-nginx` and a file called `Dockerfile` inside it.
     2. Write the following into the Dockerfile:
        ```dockerfile
        FROM nginx:latest
        COPY index.html /usr/share/nginx/html/index.html
        ```
     3. Create a simple `index.html` file in the same directory with custom content.
     4. Build the image:
        ```bash
        docker build -t custom-nginx .
        ```
     5. Run a container from the custom image:
        ```bash
        docker run -d -p 8082:80 custom-nginx
        ```
     6. Verify the content by visiting `http://localhost:8082`.

---

### 4. **Working with Docker Volumes**
   **Objective:** Understand persistent storage using Docker volumes and bind mounts.
   - **Exercise:**
     1. Create a Docker volume:
        ```bash
        docker volume create nginx-data
        ```
     2. Run an `nginx` container with the volume:
        ```bash
        docker run -d -p 8083:80 -v nginx-data:/usr/share/nginx/html nginx
        ```
     3. Create an HTML file in the volume path on your host system to update the `nginx` homepage.
     4. Refresh the browser to see the updated content at `http://localhost:8083`.

---

### 5. **Networking in Docker**
   **Objective:** Learn how Docker handles networking, including bridge, host, and custom networks.
   - **Exercise:**
     1. Create a custom Docker network:
        ```bash
        docker network create my-network
        ```
     2. Run two containers (e.g., `nginx` and `alpine`) on the same network:
        ```bash
        docker run -d --name nginx-server --network my-network nginx
        docker run -it --name alpine-client --network my-network alpine sh
        ```
     3. Inside the `alpine` container, test connectivity:
        ```bash
        ping nginx-server
        ```
     4. Verify that the `nginx` container is accessible from the `alpine` container.

---

### 6. **Docker Compose for Multi-Container Applications**
   **Objective:** Use Docker Compose to define and run multi-container applications.
   - **Exercise:**
     1. Create a `docker-compose.yml` file with the following:
        ```yaml
        version: '3'
        services:
          web:
            image: nginx
            ports:
              - "8084:80"
          client:
            image: alpine
            command: ["ping", "web"]
        ```
     2. Run the application with:
        ```bash
        docker-compose up -d
        ```
     3. Check the logs to see the `alpine` container pinging the `nginx` container:
        ```bash
        docker-compose logs
        ```

---

### 7. **Container Lifecycle Management and Troubleshooting**
   **Objective:** Understand the lifecycle of containers, including creating, starting, stopping, and troubleshooting containers.
   - **Exercise:**
     1. Run a container with an incorrect command:
        ```bash
        docker run --name error-container nginx invalidcommand
        ```
     2. Check the container status:
        ```bash
        docker ps -a
        ```
     3. View the error logs to identify the issue:
        ```bash
        docker logs error-container
        ```
     4. Remove the container:
        ```bash
        docker rm error-container
        ```

---

### 8. **Docker Registries: Storing and Sharing Images**
   **Objective:** Push and pull images to/from Docker Hub and private registries.
   - **Exercise:**
     1. Tag your custom `nginx` image:
        ```bash
        docker tag custom-nginx yourusername/custom-nginx:latest
        ```
     2. Log in to Docker Hub:
        ```bash
        docker login
        ```
     3. Push the image to Docker Hub:
        ```bash
        docker push yourusername/custom-nginx:latest
        ```
     4. Pull the image from Docker Hub:
        ```bash
        docker pull yourusername/custom-nginx:latest
        ```

---

### 9. **Container Orchestration Basics with Docker Swarm**
   **Objective:** Set up a simple Docker Swarm, deploy services, and understand scaling.
   - **Exercise:**
     1. Initialize a Swarm:
        ```bash
        docker swarm init
        ```
     2. Deploy a service:
        ```bash
        docker service create --name nginx-swarm -p 8085:80 nginx
        ```
     3. Scale the service to 3 replicas:
        ```bash
        docker service scale nginx-swarm=3
        ```
     4. Verify the replicas:
        ```bash
        docker service ps nginx-swarm
        ```

---

### 10. **Securing Docker: Security Best Practices**
   **Objective:** Implement basic security best practices for Docker.
   - **Exercise:**
     1. Pull the `alpine` image to create a secure, minimal container:
        ```bash
        docker pull alpine
        ```
     2. Run a container in read-only mode:
        ```bash
        docker run -it --read-only alpine sh
        ```
     3. Try creating a file to confirm that the filesystem is read-only.
     4. Exit the container and explore how to use user namespaces and `docker scan` for vulnerability detection on images:
        ```bash
        docker scan alpine
        ```

---

