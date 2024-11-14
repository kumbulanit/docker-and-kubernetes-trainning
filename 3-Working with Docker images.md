

### 1. **Pulling Images from Docker Hub**

Docker Hub is a public repository where many popular images are hosted. You can pull images from Docker Hub using the `docker pull` command.

- **Example**: Pulling the latest `nginx` image from Docker Hub:
   ```bash
   docker pull nginx
   ```
   This command downloads the latest `nginx` image locally.

- **Specify a Version**: You can specify a version by adding a tag (e.g., `:1.21`).
   ```bash
   docker pull nginx:1.21
   ```

---

### 2. **Building Images from a Dockerfile**

You can create your own custom images by writing a `Dockerfile`, a script that contains instructions for building the image.

**Sample Dockerfile**:
```dockerfile
# Use an official base image
FROM ubuntu:20.04

# Install dependencies
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip

# Copy application code
COPY . /app

# Set the working directory
WORKDIR /app

# Install Python dependencies
RUN pip3 install -r requirements.txt

# Command to run the application
CMD ["python3", "app.py"]
```

- **Example (Build an Image)**:
   ```bash
   docker build -t my-python-app .
   ```
   This builds an image named `my-python-app` using the Dockerfile in the current directory (`.`).

---

### 3. **Listing Local Images**

To view all images stored locally, use the `docker images` command.

- **Example**:
   ```bash
   docker images
   ```
   This lists all images, showing details like repository, tag, image ID, and size.

---

### 4. **Running a Container from an Image**

To start a container from an image, use the `docker run` command. If the image isn’t available locally, Docker will pull it automatically.

- **Example**:
   ```bash
   docker run -d --name my-nginx -p 8080:80 nginx
   ```
   This starts a container named `my-nginx` from the `nginx` image and maps port 8080 on the host to port 80 in the container.

---

### 5. **Tagging Images**

Tagging allows you to give your image a unique name and version. This is particularly useful when pushing images to a repository.

- **Example (Tagging an Image)**:
   ```bash
   docker tag my-python-app myrepo/my-python-app:v1.0
   ```
   This tags the `my-python-app` image with the repository `myrepo/my-python-app` and the tag `v1.0`.

---

### 6. **Pushing Images to Docker Hub**

Once you’ve tagged an image, you can push it to Docker Hub or another registry. Ensure you’re logged in with `docker login`.

- **Example**:
   ```bash
   docker push myrepo/my-python-app:v1.0
   ```
   This pushes the tagged image `myrepo/my-python-app:v1.0` to Docker Hub.

---

### 7. **Inspecting Images**

To view detailed metadata about an image, use `docker inspect`. This command provides information about an image's layers, environment variables, exposed ports, and more.

- **Example**:
   ```bash
   docker inspect nginx
   ```
   This shows detailed JSON metadata for the `nginx` image.

---

### 8. **Removing Images**

To delete an image that’s no longer needed, use `docker rmi`. If an image is being used by a container, stop and remove the container first.

- **Example**:
   ```bash
   docker rmi my-python-app
   ```
   This removes the `my-python-app` image.

---

### 9. **Image Cleanup**

Over time, images, containers, and other Docker artifacts can consume significant disk space. Use the `docker system prune` command to remove unused data.

- **Example (Remove unused images, containers, networks, and build cache)**:
   ```bash
   docker system prune
   ```
   This command clears all unused Docker resources.

---

### 10. **Saving and Loading Images**

You can save an image to a file and load it later or transfer it to another system.

- **Save an Image**:
   ```bash
   docker save -o my-python-app.tar my-python-app
   ```
   This saves the `my-python-app` image to a `my-python-app.tar` file.

- **Load an Image**:
   ```bash
   docker load -i my-python-app.tar
   ```
   This loads the image from the `my-python-app.tar` file.

---

### 11. **Dockerfile Best Practices**

When creating Docker images, follow these best practices to make images smaller, faster, and more maintainable:
- **Use Official Base Images**: Start with official, minimal base images (e.g., `alpine`, `ubuntu`).
- **Minimize Layers**: Combine commands when possible (e.g., `RUN apt-get update && apt-get install -y ...`).
- **Avoid Storing Secrets**: Never store passwords or secrets in the Dockerfile.
- **Use Multi-Stage Builds**: For production, use multi-stage builds to reduce image size by separating the build environment from the runtime environment.

---

### Summary of Docker Image Commands

| **Command**                           | **Description**                                             |
|---------------------------------------|-------------------------------------------------------------|
| `docker pull <image>`                 | Download an image from a repository                         |
| `docker build -t <name> .`            | Build an image from a Dockerfile in the current directory   |
| `docker images`                       | List all local images                                       |
| `docker tag <image> <repo:tag>`       | Tag an image with a new name                                |
| `docker push <repo:tag>`              | Push an image to a repository                               |
| `docker inspect <image>`              | Display detailed information about an image                 |
| `docker rmi <image>`                  | Remove an image                                             |
| `docker system prune`                 | Remove unused Docker data                                   |
| `docker save -o <file> <image>`       | Save an image to a file                                     |
| `docker load -i <file>`               | Load an image from a file                                   |

---

