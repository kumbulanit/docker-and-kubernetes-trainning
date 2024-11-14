


### 1. **Creating a Container**

The first step is creating a container from an image. This can be done with the `docker create` or `docker run` command. The difference is that `docker create` just creates the container without starting it, while `docker run` creates and starts it immediately.

- **Example (Create without starting):**
   ```bash
   docker create --name my-container nginx
   ```
   This creates a container named `my-container` from the `nginx` image, but it doesn’t start it.

- **Example (Create and start):**
   ```bash
   docker run --name my-container -d nginx
   ```
   This creates and starts a container in detached mode (`-d`) based on the `nginx` image.

---

### 2. **Starting a Container**

A container that has been created can be started at any time with the `docker start` command.

- **Example:**
   ```bash
   docker start my-container
   ```
   This starts the `my-container` container.

---

### 3. **Stopping a Container**

The `docker stop` command gracefully stops a running container, allowing it to finish any in-progress work and shut down cleanly.

- **Example:**
   ```bash
   docker stop my-container
   ```
   This stops `my-container` gracefully.

---

### 4. **Restarting a Container**

The `docker restart` command stops and then immediately restarts a container.

- **Example:**
   ```bash
   docker restart my-container
   ```
   This restarts `my-container`, which is useful when applying configuration changes.

---

### 5. **Pausing and Unpausing a Container**

Sometimes, you may want to temporarily pause all processes inside a container (e.g., for maintenance or resource management). The `docker pause` and `docker unpause` commands can do this.

- **Pause a container:**
   ```bash
   docker pause my-container
   ```
   This pauses all processes inside `my-container`.

- **Unpause a container:**
   ```bash
   docker unpause my-container
   ```
   This resumes all processes in `my-container`.

---

### 6. **Viewing Container Status**

The `docker ps` command is essential to view the status and list of containers.

- **Example (List running containers):**
   ```bash
   docker ps
   ```
   This shows all running containers.

- **Example (List all containers, including stopped ones):**
   ```bash
   docker ps -a
   ```
   This shows all containers, whether they are running, stopped, or exited.

---

### 7. **Inspecting a Container**

The `docker inspect` command provides detailed information about a container’s configuration, network settings, volume mounts, and more.

- **Example:**
   ```bash
   docker inspect my-container
   ```
   This displays detailed JSON output for `my-container`.

---

### 8. **Removing a Container**

A container that’s no longer needed can be removed using the `docker rm` command. Note that only stopped containers can be removed directly; if it’s still running, you’ll need to stop it first or use the `-f` (force) option.

- **Example (Remove a stopped container):**
   ```bash
   docker rm my-container
   ```
   This removes `my-container`.

- **Example (Force remove a running container):**
   ```bash
   docker rm -f my-container
   ```
   This forcibly stops and removes `my-container`.

---

### 9. **Container Cleanup**

Docker provides a few commands to help clean up unused containers, images, networks, and volumes, which accumulate over time.

- **Example (Remove all stopped containers):**
   ```bash
   docker container prune
   ```
   This command removes all containers that are stopped, helping to free up space.

---

### 10. **Container Logs**

The `docker logs` command is used to view the logs of a container, which is useful for troubleshooting and monitoring.

- **Example:**
   ```bash
   docker logs my-container
   ```
   This displays the logs for `my-container`.

---

### Summary of Lifecycle Commands

| **Lifecycle Stage**     | **Command**                             |
|-------------------------|-----------------------------------------|
| Create                  | `docker create`, `docker run`           |
| Start                   | `docker start`                          |
| Stop                    | `docker stop`                           |
| Restart                 | `docker restart`                        |
| Pause                   | `docker pause`                          |
| Unpause                 | `docker unpause`                        |
| Status                  | `docker ps`, `docker inspect`           |
| Remove                  | `docker rm`, `docker container prune`   |
| View Logs               | `docker logs`                           |

---
