# 🚀 Personal Flask API

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-2.x-000000?style=for-the-badge&logo=flask&logoColor=white)
![Gunicorn](https://img.shields.io/badge/Gunicorn-WSGI-499848?style=for-the-badge&logo=gunicorn&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-Reverse%20Proxy-009639?style=for-the-badge&logo=nginx&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Status](https://img.shields.io/badge/Status-Live-brightgreen?style=for-the-badge)
![License](https://img.shields.io/badge/License-Educational-blue?style=for-the-badge)

---

## 📌 Project Overview

A lightweight **Personal REST API** built with Python/Flask and deployed on AWS EC2 using a production-grade DevOps stack. This project demonstrates a real-world deployment pipeline — from writing an API to keeping it persistently alive behind a reverse proxy.

> **Live URL:** `http://13.60.222.90`

---

## 🏗️ Architecture Diagram

```
                        ┌─────────────────────────────────────────┐
                        │              AWS EC2 Instance            │
                        │              (Ubuntu 22.04)              │
                        │                                          │
  Client Request        │   ┌──────────┐       ┌──────────────┐   │
 ──────────────────────►│   │          │ :5000 │              │   │
  http://your-ip:80     │   │  Nginx   ├──────►│  Gunicorn    │   │
                        │   │  (Port   │       │  (WSGI       │   │
  ◄──────────────────── │   │   80)    │◄──────┤   Server)    │   │
  JSON Response         │   │          │       │              │   │
                        │   └──────────┘       └──────┬───────┘   │
                        │   Reverse Proxy              │           │
                        │                       ┌──────▼───────┐   │
                        │                       │  Flask App   │   │
                        │                       │  (app.py)    │   │
                        │                       └──────────────┘   │
                        │                                          │
                        │   ┌──────────────────────────────────┐   │
                        │   │  systemd — keeps service alive   │   │
                        │   │  auto-restarts on crash/reboot   │   │
                        │   └──────────────────────────────────┘   │
                        └─────────────────────────────────────────┘

  Stack:  Python/Flask → Gunicorn → Nginx → Internet
  Port:   Flask/Gunicorn binds to 127.0.0.1:5000 (private)
          Nginx listens on 0.0.0.0:80 (public)
```

---

## 📡 API Endpoints

| Method | Endpoint  | Description       | Status Code | Content-Type     |
|--------|-----------|-------------------|-------------|------------------|
| GET    | `/`       | API status check  | `200 OK`    | application/json |
| GET    | `/health` | Health check      | `200 OK`    | application/json |
| GET    | `/me`     | Personal details  | `200 OK`    | application/json |

### Expected Responses

**GET /**
```json
{
  "message": "API is running"
}
```

**GET /health**
```json
{
  "message": "healthy"
}
```

**GET /me**
```json
{
  "name": "Emmanuel Kabari",
  "email": "kabariirenaeus@gmail.com",
  "github": "https://github.com/Kabari"
}
```

---

## 🗂️ Project File Structure

```
personal-api/
│
├── app.py               # Main Flask application (all 3 endpoints)
├── requirements.txt     # Python dependencies (Flask, Gunicorn)
├── README.md            # Project documentation (you are here)
│
└── venv/                # Python virtual environment (not committed to Git)
    ├── bin/
    │   ├── python3
    │   └── gunicorn
    └── lib/
```

---

## 🧑‍💻 Run Locally

### Prerequisites
- Python 3.10 or higher
- pip
- git

### Steps

**1. Clone the repository**
```bash
git clone https://github.com/Kabari/personal-api.git
cd personal-api
```

**2. Create and activate a virtual environment**
```bash
python3 -m venv venv
source venv/bin/activate        # Mac/Linux
# venv\Scripts\activate         # Windows
```

**3. Install dependencies**
```bash
pip install -r requirements.txt
```

**4. Run the application**
```bash
python3 app.py
```

**5. Test all endpoints**
```bash
curl http://127.0.0.1:5000/
curl http://127.0.0.1:5000/health
curl http://127.0.0.1:5000/me
```

---

## ☁️ Deployment Guide (AWS EC2 + Nginx)

### 1. Provision the Server

- Launch an **AWS EC2** instance (Ubuntu 22.04 LTS, t2.micro)
- Configure inbound Security Group rules:

| Port | Protocol | Source    | Purpose        |
|------|----------|-----------|----------------|
| 22   | TCP      | Your IP   | SSH access     |
| 80   | TCP      | 0.0.0.0/0 | HTTP traffic   |
| 443  | TCP      | 0.0.0.0/0 | HTTPS (future) |

> ⚠️ Do **NOT** open port 5000 publicly. Flask/Gunicorn must stay private.

---

### 2. Connect via SSH

```bash
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@your-server-ip
```

---

### 3. Install System Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-pip python3-venv nginx -y
```

---

### 4. Set Up the Project

```bash
mkdir ~/personal-api && cd ~/personal-api
python3 -m venv venv
source venv/bin/activate
pip install flask gunicorn
pip freeze > requirements.txt
```

Create `app.py`:

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/")
def home():
    return jsonify({"message": "API is running"}), 200

@app.route("/health")
def health():
    return jsonify({"message": "healthy"}), 200

@app.route("/me")
def me():
    return jsonify({
        "name": "Emmanuel Kabari",
        "email": "kabariirenaeus@gmail.com",
        "github": "https://github.com/Kabari"
    }), 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

---

### 5. Configure systemd (Persistent Process Manager)

```bash
sudo nano /etc/systemd/system/personal-api.service
```

Paste:

```ini
[Unit]
Description=Personal Flask API
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/personal-api
Environment="PATH=/home/ubuntu/personal-api/venv/bin"
ExecStart=/home/ubuntu/personal-api/venv/bin/gunicorn -w 3 -b 127.0.0.1:5000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl start personal-api
sudo systemctl enable personal-api
```

---

### 6. Configure Nginx Reverse Proxy

```bash
sudo nano /etc/nginx/sites-available/personal-api
```

Paste:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Enable and restart:

```bash
sudo ln -s /etc/nginx/sites-available/personal-api /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl restart nginx
```

---

## ✅ Verification Commands

Run these to confirm everything is working correctly:

```bash
# 1. Check the service is running
sudo systemctl status personal-api

# 2. Check Nginx is running
sudo systemctl status nginx

# 3. Test endpoints internally (on the server)
curl http://127.0.0.1:5000/
curl http://127.0.0.1:5000/health
curl http://127.0.0.1:5000/me

# 4. Test endpoints publicly (from your local machine)
curl -i http://your-server-ip/
curl -i http://your-server-ip/health
curl -i http://your-server-ip/me

# 5. Verify Content-Type header is application/json
curl -I http://your-server-ip/

# 6. Check response time (must be under 500ms)
curl -o /dev/null -s -w "Total time: %{time_total}s\n" http://your-server-ip/
```

---

## 🔒 Security Checklist

```
✅  Flask app binds to 127.0.0.1:5000 (not exposed publicly)
✅  Only ports 22 and 80 open in AWS Security Group
✅  Port 5000 is NOT open in AWS Security Group
✅  Nginx acts as the only public entry point (port 80)
✅  App runs as non-root user (ubuntu)
✅  systemd auto-restarts app on crash or server reboot
✅  All endpoints return Content-Type: application/json
✅  All endpoints return HTTP 200
✅  All endpoints respond within 500ms
✅  GitHub repository is set to Public
```

---

## 🛠️ Troubleshooting

**Service not running?**
```bash
sudo journalctl -u personal-api -n 50
sudo systemctl restart personal-api
```

**502 Bad Gateway from Nginx?**
```bash
# Gunicorn is likely not running
sudo systemctl status personal-api
sudo systemctl start personal-api
```

**Nginx config error?**
```bash
sudo nginx -t
sudo systemctl restart nginx
```

**Getting HTML instead of JSON?**
```bash
# Default Nginx page is still active
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
```

**Check live logs in real time:**
```bash
sudo journalctl -u personal-api -f
```

---

## 📚 Tech Stack

| Technology | Role                          |
|------------|-------------------------------|
| Python 3   | Programming language          |
| Flask      | Web framework / API           |
| Gunicorn   | Production WSGI server        |
| Nginx      | Reverse proxy / load balancer |
| systemd    | Process & service manager     |
| AWS EC2    | Cloud server (VPS)            |
| Ubuntu     | Server operating system       |

---

## 👤 Author

**Emmanuel Kabari**
📧 kabariirenaeus@gmail.com
🐙 [https://github.com/Kabari](https://github.com/Kabari)

---

