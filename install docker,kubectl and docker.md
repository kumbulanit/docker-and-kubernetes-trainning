To install Docker, Minikube, and kubectl on Ubuntu 22.04, follow these steps:

---

### Step 1: Install Docker
1. **Update and Install Required Packages**:
    ```bash
    sudo apt update
    sudo apt install -y ca-certificates curl gnupg lsb-release
    ```

2. **Add Docker’s Official GPG Key**:
    ```bash
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    ```

3. **Set Up Docker Repository**:
    ```bash
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

4. **Install Docker Engine**:
    ```bash
    sudo apt update
    sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

5. **Start and Enable Docker**:
    ```bash
    sudo systemctl start docker
    sudo systemctl enable docker
    ```

6. **Add Your User to the Docker Group** (optional):
    ```bash
    sudo usermod -aG docker $USER
    ```
    Log out and back in or run `newgrp docker` to apply the group changes.

---

### Step 2: Install kubectl
1. **Download the Latest Release of kubectl**:
    ```bash
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    ```

2. **Install kubectl**:
    ```bash
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    ```

3. **Verify Installation**:
    ```bash
    kubectl version --client
    ```

---

### Step 3: Install Minikube
1. **Download the Latest Minikube Release**:
    ```bash
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    ```

2. **Install Minikube**:
    ```bash
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    ```

3. **Verify Installation**:
    ```bash
    minikube version
    ```

---

### Optional Step: Start Minikube
Once everything is installed, you can start Minikube with Docker as the driver:

```bash
minikube start --driver=docker
```

