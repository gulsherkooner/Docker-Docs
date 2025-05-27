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



