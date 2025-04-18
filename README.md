
# E-Commerce Application Deployment On AWS EC2

# 🛒 E-commerce Project Deployment Guide

This guide walks you through the steps to deploy your E-commerce project (Vite + React frontend, Node.js backend) on an AWS EC2 Ubuntu server using NGINX, PM2, and HTTPS (optional).

---

## 🚀 High-Level Deployment Steps:

1. Launch EC2 Ubuntu Instance  
2. Connect to EC2 via SSH  
3. Install necessary packages: Node.js, PM2, NGINX, Git  
4. Clone your project to the EC2 server  
5. Setup and run backend (Node.js API on port 8080)  
6. Build frontend (Vite + React)  
7. Configure NGINX to serve frontend and proxy API requests  
8. Secure server (optional): UFW, HTTPS with Certbot  

---

## 💍 1. Create Ubuntu Server on AWS EC2

You’re essentially creating a virtual machine (VM) in the cloud using EC2 (Elastic Compute Cloud).

- Choose an AMI: **Ubuntu 18.04**
- Instance Type: **t2.micro** (free-tier eligible)
- Create/download a **key pair (.pem file)** for SSH
- Configure **Security Group** to allow:
  - **SSH (port 22)**
  - **HTTP (port 80)**

---

## 🔐 2. Connect via SSH

```bash
chmod 400 my-key.pem
ssh -i my-key.pem ubuntu@<your-ec2-public-dns>
```

---

## 📁 3. Clone Your Project

```bash
cd /opt
sudo git clone https://github.com/Mansi-298/Ecommerse_final_deployed.git ecommerse-app
cd ecommerse-app
```

---

## 🛠️ 4. Install Node.js, PM2, NGINX, Git

```bash
# Update and install dependencies
sudo apt update
sudo apt install -y nginx git curl

# Install Node.js (LTS)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install PM2 (Process Manager)
sudo npm install -g pm2
```

---

## 🔁 5. Start Backend Server with PM2

We never want to run node directly in production. PM2 handles restarting the app if it crashes.

```bash
pm2 start /home/ubuntu/opt/ecommerse-app/server/server.js --name ecommerse-app
```

Configure PM2 to start on reboot:

```bash
pm2 startup
# Output will include a command like below - copy-paste that
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu

pm2 status
pm2 save
```

---

## 🔪 6. Setup and Run the Server (Backend)

```bash
cd server
sudo npm install
sudo pm2 start index.js --name lms-api
```

---

## 🖼️ 7. Build the Client (Frontend)

```bash
cd ../client
sudo npm install
sudo npm run build
```

The Vite build output will be in the `dist/` directory (used for NGINX serving).

Verify the build folder:

```bash
cd build
ls
```

Expected files:

```text
asset-manifest.json  favicon.ico  index.html  logo192.png  logo512.png
manifest.json  precache-manifest.js  robots.txt  service-worker.js  static
```

---

## 🌐 8. Configure NGINX

### Install and Enable NGINX

```bash
sudo apt install nginx -y
sudo systemctl enable nginx
```

### Configure NGINX Server Block

```bash
cd /etc/nginx/sites-available
sudo rm default
sudo nano default
```

Paste the following configuration:

```nginx
server {
    listen 80;
    listen [::]:80;

    root /opt/ecommerse-app/client/dist;
    index index.html;
    server_name _;

    location / {
        try_files $uri /index.html;
    }

    location /api {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable this configuration:

```bash
sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

Visit your app in the browser:  
`http://<your-ec2-public-dns>`

---

## 🔧 10. Set Up Environment Variables

Create a file (e.g., `.env`) outside the app directory:

```bash
nano /home/ubuntu/.env
```

Example content:
```env
PORT=5000
MONGO_URI=your_mongo_connection_string
JWT_SECRET=your_jwt_secret
```

Make sure environment variables are loaded at login:

```bash
nano /home/ubuntu/.profile
```

Add this line at the bottom:
```bash
set -o allexport; source /home/ubuntu/.env; set +o allexport
```

For changes to take effect:
```bash
source ~/.profile
```

Verify:
```bash
printenv
```

---

## 🛡️ 11. Enable Firewall (UFW)

```bash
sudo ufw status
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw enable
sudo ufw status
```

---

## 🔐 12. (Optional) Secure Your Server

- Set up **HTTPS** using Let's Encrypt and Certbot
- Configure domain pointing (if available)

For now, HTTP is sufficient. Your app should be accessible at:
```
http://<your-ec2-public-dns>
```

---

## Refer Star Repo To Understand more in detail
