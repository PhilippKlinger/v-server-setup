# V-Server Setup

Use this guide to configure a basic V-Server with SSH key authentication, disabled password login, NGINX, Git, and GitHub SSH access.

## Table of Contents

- [1. SSH Setup](#1-ssh-setup)
- [2. Disable Password Login](#2-disable-password-login)
- [3. Install and Test NGINX](#3-install-and-test-nginx)
- [4. Alternative NGINX Site on Port 8081](#4-alternative-nginx-site-on-port-8081)
- [5. Git Configuration](#5-git-configuration)
- [6. GitHub SSH Access from the Server](#6-github-ssh-access-from-the-server)
- [7. Validation and Testing](#7-validation-and-testing)

---

## 1. SSH Setup

Generate an SSH key pair on your local machine.

```bash
ssh-keygen -t ed25519
```

Store the key in the default location unless you want to use a custom path.

Copy your public key to the server user account.

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub <username>@<server-ip>
```

This adds your public key to the server user's `authorized_keys` file.

Test SSH login with your private key before changing the server configuration.

```bash
ssh -i ~/.ssh/id_ed25519 <username>@<server-ip>
```

---

## 2. Disable Password Login

Open the SSH server configuration.

```bash
sudo nano /etc/ssh/sshd_config
```

Set the following options:

```text
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

Save the file, then reload the SSH service.

```bash
sudo systemctl reload ssh
```

If your system uses `sshd`, use:

```bash
sudo systemctl reload sshd
```

Test SSH login again in a new terminal session.

To verify that password login is disabled, run:

```bash
ssh -o PubkeyAuthentication=no <username>@<server-ip>
```

This login attempt should fail.

---

## 3. Install and Test NGINX

Update the package index and install NGINX.

```bash
sudo apt update
sudo apt install nginx -y
```

Validate the NGINX configuration.

```bash
sudo nginx -t
```

Open your server IP in the browser to confirm that NGINX is reachable.

---

## 4. Alternative NGINX Site on Port 8081

Create a new web root for the alternative site.

```bash
sudo mkdir -p /var/www/alternatives
```

Create a new HTML file.

```bash
sudo nano /var/www/alternatives/alternate-index.html
```

Add the following HTML content:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>V-Server Setup</title>
    <style>
      body {
        font-family: Arial, sans-serif;
        background: #f4f4f4;
        color: #222;
        text-align: center;
        margin-top: 10%;
      }
      .box {
        background: white;
        max-width: 700px;
        margin: auto;
        padding: 2rem;
        border-radius: 10px;
        box-shadow: 0 0 12px rgba(0, 0, 0, 0.1);
      }
      h1 {
        color: #0a66c2;
      }
      code {
        background: #eee;
        padding: 0.2rem 0.4rem;
        border-radius: 4px;
      }
    </style>
  </head>
  <body>
    <div class="box">
      <h1>V-Server successfully configured</h1>
      <p>This server is running with <strong>NGINX</strong>.</p>
      <p>
        Custom start page location:
        <code>/var/www/alternatives/alternate-index.html</code>
      </p>
    </div>
  </body>
</html>
```

Create a separate NGINX site configuration.

```bash
sudo nano /etc/nginx/sites-enabled/alternatives
```

Add the following server block:

```nginx
server {
    listen 8081;
    listen [::]:8081;

    root /var/www/alternatives;
    index alternate-index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Validate the configuration.

```bash
sudo nginx -t
```

Reload NGINX.

```bash
sudo systemctl reload nginx
```

Open the alternative site in your browser:

```text
http://<server-ip>:8081
```

This setup keeps the default NGINX site unchanged and serves the custom page on port `8081`.

---

## 5. Git Configuration

Set your Git username and email address on the server.

```bash
git config --global user.name "YOUR NAME"
git config --global user.email "YOUR_EMAIL@example.com"
```

Verify the configuration.

```bash
git config --global --list
```

---

## 6. GitHub SSH Access from the Server

Generate a separate SSH key pair on the server for GitHub access.

```bash
ssh-keygen -t ed25519 -C "YOUR_EMAIL@example.com"
```

Display the public key.

```bash
cat ~/.ssh/id_ed25519.pub
```

Add the public key to your GitHub account under **Settings > SSH and GPG keys**.

Test the connection.

```bash
ssh -T git@github.com
```

Use this key to clone and pull repositories from GitHub over SSH.

---

## 7. Validation and Testing

Validate the SSH setup:

- SSH login with a key works
- Password-based login is disabled

Validate the NGINX setup:

```bash
sudo nginx -t
```

Open the alternative site in your browser:

```text
http://<server-ip>:8081
```

Validate the Git setup:

- Git username and email are configured
- GitHub SSH authentication works from the server
