# Deploy to a VPS (generic concepts)

How to take an app (FastAPI, but this applies to any backend) from "runs on my laptop" to "runs on a real server, with a domain and HTTPS." The steps are generic — they work the same on Akamai/Linode, DigitalOcean, Hetzner, AWS EC2, etc. It's Linux config, not provider-specific.

## 1. SSH with keys instead of a password

A fresh VPS lets you log in via SSH with a username+password by default — that's brute-forceable from anywhere in the world. The alternative is an asymmetric key pair: the private key stays on your machine, the public key travels to the server. No one can get in without the private key, and that one never leaves your disk.

```bash
# on your local machine
ssh-keygen -t ed25519 -C "your_email@example.com"
ssh-copy-id user@server-ip

# on the server, /etc/ssh/sshd_config
PasswordAuthentication no
PermitRootLogin no

sudo systemctl restart sshd
```

## 2. Firewall (ufw)

The server has every port potentially reachable from the internet. A firewall is an explicit whitelist: you only let through what you actually need (SSH, HTTP, HTTPS), and everything else is dropped before it reaches any process.

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

## 3. Brute-force protection (fail2ban)

Even with password login disabled, bots keep scanning IPs and trying logins against port 22. `fail2ban` reads the auth logs, detects repeated failed attempts from the same IP, and bans it at the firewall level for a while. It's a second layer, not a replacement for the SSH key auth from step 1.

```bash
sudo apt install fail2ban
sudo systemctl enable --now fail2ban
```

## 4. Nginx as reverse proxy

Your app (`uvicorn`/`fastapi run`) listens on an internal port, **not directly exposed to the internet** — for example `127.0.0.1:8000`. Nginx listens on the public ports (80/443) and forwards traffic to your app. The advantage of not exposing uvicorn directly: Nginx handles TLS, compression, static files, and rate limiting without your code having to know anything about it, and you can run multiple apps on the same server by domain.

```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name myapp.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## 5. HTTPS with Let's Encrypt

An HTTPS certificate has to be signed by a trusted authority, or the browser throws a warning. Let's Encrypt issues them for free, and `certbot` automates both issuance and renewal (they expire every ~90 days). Certbot rewrites the Nginx config so TLS terminates there: it decrypts the public HTTPS and passes plain HTTP to your app internally.

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d myapp.com
```

## 6. systemd to run the app as a service

If you run `fastapi run` by hand in an SSH session, the process dies as soon as you close that session. A systemd `.service` runs it as a daemon: it starts automatically when the server boots, and restarts it automatically if it crashes — the same mechanism the operating system itself uses for its internal services.

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=FastAPI app
After=network.target

[Service]
User=deploy
WorkingDirectory=/home/deploy/myapp
ExecStart=/home/deploy/myapp/.venv/bin/fastapi run main.py --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now myapp
sudo systemctl status myapp
```

---
Related: [Sync vs Async in FastAPI](../stacks/fastapi/sync-vs-async.md), [Vertical vs Horizontal Scalability](scaling-vertical-vs-horizontal.md), [VPS vs Cloud Run](vps-vs-cloud-run.md).
