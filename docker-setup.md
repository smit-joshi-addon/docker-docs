### 1. **Uninstall Old Versions (if any):**
   First, remove any older versions of Docker in case they’re causing issues:
   ```bash
   sudo apt-get remove docker docker-engine docker.io containerd runc
   ```

### 2. **Update the Package Index:**
   Ensure your package list is up-to-date:
   ```bash
   sudo apt-get update
   ```

### 3. **Install Dependencies:**
   These are required for apt to use HTTPS:
   ```bash
   sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
   ```

### 4. **Add Docker's GPG Key:**
   This step is crucial for authenticating Docker packages:
   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

### 5. **Add Docker Repository:**
   Ensure you add the repository for Docker:
   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

### 6. **Update the Package List Again:**
   Refresh the list of available packages from the newly added repository:
   ```bash
   sudo apt-get update
   ```

### 7. **Install Docker Engine:**
   Now, install Docker:
   ```bash
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

### 8. **Check Docker Version:**
   Confirm that Docker has been installed successfully by checking the version:
   ```bash
   docker --version
   ```

### 9. **Post-Installation Steps:**
   If Docker is installed but you are having issues running it without `sudo`, add your user to the `docker` group:
   ```bash
   sudo usermod -aG docker $USER
   ```

   Then log out and back in or run:
   ```bash
   newgrp docker
   ```

---