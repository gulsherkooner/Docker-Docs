# Docker-Docs

**Generate SSH Key on Windows:**

Open PowerShell:

run: 
``` bash 
ssh-keygen -t ed25519 -C "ubuntu-4gb-hel1-2"
```

Press Enter for default file location

Press Enter twice to skip passphrase (or set one for extra security).

Output: 
```bash
Your public key has been saved in C:\Users\uset\.ssh\id_ed25519.pub
```

View the public key: 
```bash
Get-Content C:\Users\uset\.ssh\id_ed25519.pub
```

Copy the output (e.g., ssh-ed25519 AAAAC3Nza... ubuntu-4gb-hel1-2).

**Add SSH Key to Hetzner:**

In Hetzner Cloud Console, go to your project > Security > SSH Keys.

Click Add SSH Key.

Paste the public key from step 3.

Name it (e.g., ubuntu-4gb-hel1-2).

Click Add.

**Test SSH Access from Windows:**

In PowerShell, connect to the server:
```
ssh root@135.181.192.55
```


If prompted, type yes to accept the host key.
Expected output: 
```
Ubuntu terminal (e.g., root@cent-stage-server:~#).
```

run
```
apt update && apt upgrade -y
```
```
exit
```


**Set Up Firewall (Optional but Recommended):**

In Hetzner Cloud Console, go to Firewalls > Create Firewall.

Name: cent-stage-firewall.

Add rules:

-->SSH: Protocol: TCP, Port: 22, Source: 0.0.0.0/0.

-->HTTP: Protocol: TCP, Port: 80, Source: 0.0.0.0/0.

-->HTTPS: Protocol: TCP, Port: 443, Source: 0.0.0.0/0.

***apply rules for both inbound and outbound***

Apply to your server: Select ubuntu-4gb-hel1-2.

Click Create.

This allows SSH and web traffic while blocking other ports.


**Install Docker and Docker Compose on Hetzner Server**
```
ssh root@135.181.192.55
```

Install Docker:
```
sudo apt update
```
install prerequisites:
```
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```
Add Docker’s GPG key:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
Add Docker repository:
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Update package index again:
```
sudo apt update
```
Install Docker:
```
sudo apt install -y docker-ce docker-ce-cli containerd.io
```
Verify Docker installation:
```
sudo docker --version
```
Start and enable Docker service:
```
sudo systemctl start docker
sudo systemctl enable docker
```
Check Docker status:
```
sudo systemctl status docker
```
Install Docker Compose:
```
sudo curl -L "https://github.com/docker/compose/releases/download/v2.36.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
Make it executable:
```
sudo chmod +x /usr/local/bin/docker-compose
```
Verify installation:
```
docker-compose --version
```

**Allow Non-Root Docker Usage (Optional but Recommended):**

Add the root user to the docker group to run Docker without sudo:
```
sudo usermod -aG docker $USER
```
Log out and reconnect via SSH to apply:
```
exit
ssh root@135.181.192.55
```
Test Docker without sudo:
```
docker ps
```


**Test Docker with a Sample Container:**

Run a test container to verify Docker:
```
docker run --rm hello-world
```
Expected output: Message starting with Hello from Docker!.

This confirms Docker is working.

Remove unused images:
```
docker image rm hello-world
```


**Open Firewall Ports for Services:**
