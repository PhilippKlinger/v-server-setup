# V-Server Setup

## Overview

This repository documents the setup of a fresh V-Server for the academy project.

The server was configured with SSH key authentication, password login was disabled, NGINX was installed, an alternative NGINX site was created on port `8081`, and Git/GitHub access was configured.

## Table of Contents

- [Overview](#overview)
- [Server Information](#server-information)
- [1. SSH Setup](#1-ssh-setup)
- [2. Disable Password Login](#2-disable-password-login)
- [3. Install and Test NGINX](#3-install-and-test-nginx)
- [4. Alternative NGINX Site on Port 8081](#4-alternative-nginx-site-on-port-8081)
- [5. Git Configuration](#5-git-configuration)
- [6. GitHub SSH Access from the Server](#6-github-ssh-access-from-the-server)
- [7. Validation and Testing](#7-validation-and-testing)

## Server Information

- **Operating System:** `Ubuntu`
- **Web Server:** `NGINX`
- **Project Repository:** `v-server-setup`

---

## 1. SSH Setup

An SSH key pair was generated on the local machine and the public key was copied to the server.

The public key was added to the server user's `authorized_keys` file using `ssh-copy-id`.

Example command:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub <username>@<server-ip>
```

After that, SSH login with the key was tested successfully.

Example login:

```bash
ssh -i ~/.ssh/id_ed25519 <username>@<server-ip>
```

---

## 2. Disable Password Login

After confirming that SSH key authentication worked correctly, password-based login was disabled in the SSH server configuration.

The SSH configuration was updated to disable password authentication and allow public key authentication.

Relevant settings:

```text
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

After the change, the SSH service was reloaded and tested again.

---

## 3. Install and Test NGINX

NGINX was installed on the server using the package manager.

Example commands:

```bash
sudo apt update
sudo apt install nginx -y
```

After installation, the service was checked and tested in the browser by opening the server IP address.

Configuration validation:

```bash
sudo nginx -t
```

---

## 4. Alternative NGINX Site on Port 8081

Instead of modifying the default NGINX site directly, an alternative site configuration was created.

### Alternative web root

```text
/var/www/alternatives
```

### Alternative HTML file

```text
/var/www/alternatives/alternate-index.html
```

### Alternative NGINX site configuration

```text
/etc/nginx/sites-enabled/alternatives
```

### NGINX server block

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

### Example HTML file

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
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
      box-shadow: 0 0 12px rgba(0,0,0,0.1);
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
    <p>Custom start page location: <code>/var/www/alternatives/alternate-index.html</code></p>
    <p>DA Project: <strong>v-server-setup</strong></p>
  </div>
</body>
</html>
```

### Result

The alternative page is available at:

```text
http://YOUR_SERVER_IP:8081
```

### Important note

The default NGINX configuration was left unchanged.

The alternative site was added as a separate configuration on port `8081`.

---

## 5. Git Configuration

Git was configured globally on the server with the same username and email address used for GitHub.

Example commands:

```bash
git config --global user.name "YOUR NAME"
git config --global user.email "YOUR_EMAIL@example.com"
```

Verification:

```bash
git config --global --list
```

---

## 6. GitHub SSH Access from the Server

A separate SSH key pair was created directly on the server to allow the server to interact with GitHub repositories.

Example command:

```bash
ssh-keygen -t ed25519 -C "YOUR_EMAIL@example.com"
```

The generated public key was then added to GitHub under **Settings > SSH and GPG keys**.

After adding the key, the connection was tested:

```bash
ssh -T git@github.com
```

This allows the server to clone and pull repositories via SSH.

---

## 7. Validation and Testing

The following tests were completed successfully.

### SSH

- Login with SSH key works
- Login with username and password does not work anymore

Example negative test for password login:

```bash
ssh -o PubkeyAuthentication=no <username>@<server-ip>
```

### NGINX

- NGINX is installed and running
- The default NGINX installation was validated with:

```bash
sudo nginx -t
```

- The alternative site is reachable at:

```text
http://YOUR_SERVER_IP:8081
```

### Git / GitHub

- Git username and email are configured
- GitHub SSH authentication from the server works
