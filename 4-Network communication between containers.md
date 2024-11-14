
### Docker Networking Types

1. **Bridge Network** (default network type): Containers can communicate with each other if they’re on the same bridge network, using container names as DNS names.
2. **Host Network**: Containers share the host’s network stack, allowing them to access the network as if they were directly on the host.
3. **Overlay Network**: Used in Docker Swarm mode for multi-host networking, connecting containers across multiple hosts.
4. **Custom User-Defined Network**: Enables more advanced configurations, such as setting aliases and connecting specific containers.

---

### 1. **Bridge Network Communication (Default)**

The default bridge network lets containers communicate via their IP addresses on the same network. Containers in the default bridge network cannot reach each other by their names directly.

**Example:**
1. Run two containers in the default bridge network:
   ```bash
   docker run -d --name container1 busybox sleep 3600
   docker run -d --name container2 busybox sleep 3600
   ```
2. Get the IP address of `container1`:
   ```bash
   docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container1
   ```
3. Connect to `container2` and ping `container1`:
   ```bash
   docker exec -it container2 sh
   ping [container1 IP address]
   ```

While this setup allows communication, the containers don’t automatically resolve each other’s names. For easier communication, consider using a **user-defined bridge network**.

---

### 2. **User-Defined Bridge Network**

User-defined bridge networks provide automatic DNS resolution, allowing containers to communicate using container names. This is more convenient for multi-container applications.

**Example:**
1. Create a user-defined network:
   ```bash
   docker network create my-network
   ```
2. Run two containers on this network:
   ```bash
   docker run -d --name web-server --network my-network nginx
   docker run -d --name client --network my-network busybox sleep 3600
   ```
3. Connect to the `client` container and communicate with `web-server` by name:
   ```bash
   docker exec -it client sh
   ping web-server
   ```
   The `web-server` container responds to `ping` because Docker’s internal DNS resolves the container name on the same network.

---

### 3. **Host Network**

When using the host network, a container shares the host’s network namespace. This mode removes network isolation, making it useful for performance-sensitive applications or when direct host network access is required.

**Example:**
1. Run a container with the host network:
   ```bash
   docker run -d --network host nginx
   ```
2. Since the container uses the host’s network, any service (e.g., `nginx` on port 80) can be accessed directly via the host’s IP address and port:
   ```bash
   curl http://localhost
   ```

Note that the host network is only available on Linux, as it doesn’t work the same way on Windows and macOS.

---

### 4. **Overlay Network for Multi-Host Communication**

The overlay network is used primarily in Docker Swarm mode, allowing containers on different hosts to communicate with each other.

**Example:**
1. Initialize Docker Swarm mode:
   ```bash
   docker swarm init
   ```
2. Create an overlay network:
   ```bash
   docker network create -d overlay my-overlay
   ```
3. Deploy services to use the overlay network:
   ```bash
   docker service create --name web --network my-overlay -p 8080:80 nginx
   docker service create --name client --network my-overlay alpine ping web
   ```
   
In this setup, the `client` service can reach the `web` service across different hosts as long as they are in the same Swarm overlay network.

---

### 5. **Container Linking (Legacy)**

While still supported, container linking is an older feature for communication and is generally replaced by user-defined networks.

To use links, specify the `--link` option when starting a container.

**Example:**
1. Run the first container:
   ```bash
   docker run -d --name db-server alpine sleep 3600
   ```
2. Start another container, linking it to the first container:
   ```bash
   docker run -it --name app-server --link db-server alpine sh
   ```
3. In `app-server`, access `db-server` via the alias name:
   ```bash
   ping db-server
   ```

---

### Docker Network Commands Cheat Sheet

| **Command**                                  | **Description**                                        |
|----------------------------------------------|--------------------------------------------------------|
| `docker network ls`                          | List all Docker networks                               |
| `docker network create <network-name>`       | Create a new user-defined network                      |
| `docker network connect <network> <container>` | Connect a container to an existing network          |
| `docker network disconnect <network> <container>` | Disconnect a container from a network             |
| `docker network inspect <network>`           | Display detailed information about a network          |
| `docker network rm <network>`                | Remove a user-defined network                          |

---

### Summary of Communication Patterns

- **Default Bridge**: Simple communication between containers via IP addresses (not names).
- **User-Defined Bridge**: Allows name-based DNS resolution, making communication easier and more manageable.
- **Host Network**: Shares the host’s network namespace; ideal for cases needing high network performance or direct host access.
- **Overlay Network**: Enables cross-host communication, primarily used in Docker Swarm for distributed applications.
- **Links (Legacy)**: An older way to link containers, mostly replaced by user-defined networks.

