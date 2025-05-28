# Docker-Docs: Setting Up a Docker Environment on Hetzner

This guide walks you through setting up SSH access, configuring a firewall, and installing Docker and Docker Compose on an Ubuntu server hosted with Hetzner Cloud.

---

## 🔑 1. Generate SSH Key on Windows

1.  **Open PowerShell.**

2.  **Run the key generation command:**
    ```powershell
    ssh-keygen -t ed25519 -C "ubuntu-4gb-hel1-2"
    ```
    *   Press **Enter** to accept the default file location (e.g., `C:\Users\your_username\.ssh\id_ed25519`).
    *   Press **Enter** twice to skip setting a passphrase (or enter a passphrase for extra security).

3.  **Output Confirmation:**
    You should see a confirmation message similar to:
    ```
    Your public key has been saved in C:\Users\uset\.ssh\id_ed25519.pub
    ```

4.  **View and Copy Your Public Key:**
    ```powershell
    Get-Content C:\Users\uset\.ssh\id_ed25519.pub
    ```
    Copy the entire output. It will look something like:
    `ssh-ed25519 AAAAC3Nza... ubuntu-4gb-hel1-2`

---

## ☁️ 2. Add SSH Key to Hetzner Cloud

1.  Navigate to the Hetzner Cloud Console.
2.  Go to your **Project > Security > SSH Keys**.
3.  Click **Add SSH Key**.
4.  **Paste** the public key you copied in the previous section.
5.  **Name** the key (e.g., `ubuntu-4gb-hel1-2`).
6.  Click **Add SSH Key**.

---

## 🧪 3. Test SSH Access from Windows

1.  **Connect to your server via PowerShell:**
    Replace `135.181.192.55` with your server's actual IP address.
    ```powershell
    ssh root@135.181.192.55
    ```

2.  If prompted to trust the host key, type `yes` and press **Enter**.

3.  **Expected Output:** You should be logged into your Ubuntu server terminal:
    ```
    root@your-server-name:~#
    ```
    (For example: `root@cent-stage-server:~#`)

4.  **Update and upgrade your server:**
    ```bash
    apt update && apt upgrade -y
    ```

5.  **Exit the SSH session:**
    ```bash
    exit
    ```

---

## 🔥 4. Set Up Firewall (Optional but Recommended)

1.  In the Hetzner Cloud Console, go to **Your Project > Security > Firewalls**.
2.  Click **Create Firewall**.
3.  **Name:** `cent-stage-firewall` (or your preferred name).
4.  **Add Rules:**
    Create the following rules. Ensure you apply them for **both inbound and outbound traffic** as per your requirement.

    | Service | Direction | Protocol | Port | Source IPs  |
    | :------ | :-------- | :------- | :--- | :---------- |
    | SSH     | Inbound   | TCP      | 22   | `0.0.0.0/0` |
    | HTTP    | Inbound   | TCP      | 80   | `0.0.0.0/0` |
    | HTTPS   | Inbound   | TCP      | 443  | `0.0.0.0/0` |
    | SSH     | Outbound  | TCP      | 22   | `0.0.0.0/0` |
    | HTTP    | Outbound  | TCP      | 80   | `0.0.0.0/0` |
    | HTTPS   | Outbound  | TCP      | 443  | `0.0.0.0/0` |

    > **Note:** Applying rules for *outbound* traffic with specific ports like this is less common. Typically, outbound traffic is allowed more broadly by default. Ensure this matches your security policy. If you only want to restrict *inbound*, only create inbound rules.

5.  **Apply to Server(s):**
    Under "Applied to", select your server (e.g., `ubuntu-4gb-hel1-2`).
6.  Click **Create Firewall**.

    This configuration allows SSH and standard web traffic while blocking other ports.

---

## 🐳 5. Install Docker and Docker Compose on Hetzner Server

1.  **Connect to your server again:**
    ```powershell
    ssh root@135.181.192.55
    ```

2.  **Update package list:**
    ```bash
    sudo apt update
    ```

3.  **Install prerequisites:**
    ```bash
    sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
    ```

4.  **Add Docker’s official GPG key:**
    ```bash
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    ```

5.  **Set up the Docker repository:**
    ```bash
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

6.  **Update package index again (with Docker repo):**
    ```bash
    sudo apt update
    ```

7.  **Install Docker Engine, CLI, and Containerd:**
    ```bash
    sudo apt install -y docker-ce docker-ce-cli containerd.io
    ```

8.  **Verify Docker installation:**
    ```bash
    sudo docker --version
    ```
    You should see the installed Docker version.

9.  **Start and enable Docker service:**
    ```bash
    sudo systemctl start docker
    sudo systemctl enable docker
    ```

10. **Check Docker service status:**
    ```bash
    sudo systemctl status docker
    ```
    Look for `active (running)`. Press `q` to exit the status view.

11. **Install Docker Compose:**
    (Check [Docker Compose releases page](https://github.com/docker/compose/releases) for the latest version and update `v2.26.1` if necessary.)
    ```bash
    DOCKER_COMPOSE_VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/') # Dynamically gets latest
    sudo curl -L "https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    ```

12. **Make Docker Compose executable:**
    ```bash
    sudo chmod +x /usr/local/bin/docker-compose
    ```

13. **Verify Docker Compose installation:**
    ```bash
    docker-compose --version
    ```
    You should see the installed Docker Compose version.

---

## 👤 6. Allow Non-Root Docker Usage (Optional but Recommended)

1.  **Add your user to the `docker` group:**
    Since you are logged in as `root`, `$USER` will be `root`. If you create a non-root user later, use that username instead of `$USER`.
    ```bash
    sudo usermod -aG docker $USER
    ```
    > **Security Note:** Adding `root` to the `docker` group doesn't change much as `root` already has privileges. This step is more impactful for non-root users. If you intend to run Docker as a non-root user, create that user first (e.g., `sudo adduser myuser`), then add `myuser` to the `docker` group (`sudo usermod -aG docker myuser`).

2.  **Apply group changes:**
    Log out and log back into the server for the group changes to take effect.
    ```bash
    exit
    ```
    Then reconnect:
    ```powershell
    ssh root@135.181.192.55
    ```
    (If you added a non-root user, log in as that user.)

3.  **Test Docker without `sudo`:**
    ```bash
    docker ps
    ```
    This command should now run without requiring `sudo`.

---

## ✅ 7. Test Docker with a Sample Container

1.  **Run the `hello-world` container:**
    ```bash
    docker run --rm hello-world
    ```

2.  **Expected Output:**
    You should see a message starting with:
    ```
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    ...
    ```
    This confirms Docker is installed and working correctly.

3.  **Clean up (optional):**
    The `--rm` flag automatically removed the container after it exited. You can remove the downloaded image if you wish:
    ```bash
    docker image rm hello-world
    ```

---

## (Open Firewall Ports for Services)

Your microservices will use specific ports (e.g., 3002 for auth-service, 3004 for post-service, 3005 for subscription-service, 5432 for PostgreSQL, 6379 for Redis, 80/443 for Nginx

1.  **Update the Hetzner firewall (set up in Step 1):** In Hetzner Cloud Console, go to Firewalls > cent-stage-firewall.
2.  Add rules:
   *    PostgreSQL: Protocol: TCP, Port: 5432, Source: 0.0.0.0/0 (restrict to server IP later for security).
   *    Redis: Protocol: TCP, Port: 6379, Source: 0.0.0.0/0 (restrict later).
   *    Custom Ports: Protocol: TCP, Port: 3000,30001,3002,3004,3005, Source: 0.0.0.0/0.
3.    Apply to cent-stage-server.

## 8.Create Deployment Directory:
Create a directory for your application on the server:
```
mkdir -p /root/cent-stage
cd /root/cent-stage
```
This will store Dockerfiles, Docker Compose, and configuration files.


##create docker containers



